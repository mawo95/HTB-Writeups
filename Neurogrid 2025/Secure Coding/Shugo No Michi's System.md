# What needs to be patched?

All possible buffer overflows inside the main file.

# Fixed Code: 

```
alles fixed? #include "env.h"
#include <libpq-fe.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

#include <string>
#include <vector>
#include <thread>
#include <mutex>
#include <fstream>
#include <sstream>
#include <chrono>
#include <atomic>
#include <filesystem>
#include <iomanip>
#include <iostream>
#include <limits>
#include <random>

#include "env.h"

#define LOG(level, msg) do { \
    auto now = std::chrono::system_clock::now(); \
    auto t = std::chrono::system_clock::to_time_t(now); \
    auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(now.time_since_epoch()) % 1000; \
    std::cout << std::put_time(std::localtime(&t), "%Y-%m-%d %H:%M:%S.") \
              << std::setfill('0') << std::setw(3) << ms.count() \
              << " [" << level << "] " << msg << std::endl; \
} while(0)

namespace fs = std::filesystem;

struct TicketRow {
    std::string name;
    std::string bus_code;
    std::string user_email;
    std::string travel_date;
    int seats = 1;
    int start_node = 0;
    int end_node = 0;
    long long total_cents = 0;
};

static std::mutex g_json_mx;
static std::string g_latest_json;
static std::random_device g_rd;
static std::mt19937 g_gen(g_rd());

static std::string jesc(const std::string &s) {
    if (s.size() > 1024 * 1024) { // 1MB limit
        return "[TRUNCATED]";
    }
    
    std::string out;
    out.reserve(s.size() + 8);
    for (unsigned char c : s) {
        switch (c) {
            case '"': out += "\\\""; break;
            case '\\': out += "\\\\"; break;
            case '\n': out += "\\n"; break;
            case '\r': out += "\\r"; break;
            case '\t': out += "\\t"; break;
            default:
                if (c < 0x20) {
                    // FIX #2: Kein sprintf mit fixed buffer mehr
                    std::ostringstream hex;
                    hex << "\\u" << std::hex 
                        << std::setw(4) << std::setfill('0')
                        << static_cast<int>(c);
                    out += hex.str();
                } else {
                    out += c;
                }
        }
    }
    return out;
}

static std::string to_json(const std::vector<TicketRow>& rows, const std::string& source) {
    std::ostringstream oss;
    oss << "{ \"source\":\"" << jesc(source)
        << "\", \"tickets\":[";
    bool first = true;
    for (const auto& r : rows) {
        if (!first) oss << ",";
        first = false;
        oss << "{"
            << "\"name\":\"" << jesc(r.name) << "\","
            << "\"bus_code\":\"" << jesc(r.bus_code) << "\","
            << "\"user_email\":\"" << jesc(r.user_email) << "\","
            << "\"travel_date\":\"" << jesc(r.travel_date) << "\","
            << "\"seats\":" << r.seats << ","
            << "\"start_node\":" << r.start_node << ","
            << "\"end_node\":" << r.end_node << ","
            << "\"total_cents\":" << r.total_cents
            << "}";
    }
    oss << "] }";
    return oss.str();
}

static void write_file_atomic(const fs::path& path, const std::string& contents) {
    try {
        fs::create_directories(path.parent_path());
        fs::path tmp = path.string() + ".tmp";
        {
            std::ofstream out(tmp, std::ios::binary);
            if (!out) {
                throw std::runtime_error("Failed to open temporary file");
            }
            out.write(contents.data(), static_cast<std::streamsize>(contents.size()));
            if (!out) {
                throw std::runtime_error("Failed to write temporary file");
            }
        }
        fs::rename(tmp, path);
    } catch (const std::exception& e) {
        LOG("ERROR", "write " << path << ": " << e.what());
        throw;
    }
}

static int safe_stoi(const char* str, int default_val) {
    if (!str || *str == '\0') {
        return default_val;
    }
    
    try {
        size_t pos;
        int result = std::stoi(str, &pos);
        if (str[pos] != '\0') {
            return default_val;
        }
        return result;
    } catch (const std::exception&) {
        return default_val;
    }
}

static long long safe_stoll(const char* str, long long default_val) {
    if (!str || *str == '\0') {
        return default_val;
    }
    
    try {
        size_t pos;
        long long result = std::stoll(str, &pos);
        if (str[pos] != '\0') {
            return default_val;
        }
        return result;
    } catch (const std::exception&) {
        return default_val;
    }
}

static std::vector<TicketRow> fetch_postgres(const Env& env) {
    std::vector<TicketRow> out;

    std::string host = env.get("PGHOST", "127.0.0.1");
    int port = env.get_int("PGPORT", 5432);
    std::string user = env.get("PGUSER", "postgres");
    std::string pass = env.get("PGPASSWORD", "");
    std::string db = env.get("PGDATABASE", "smbs_development");
    int cto = env.get_int("PGCONNECT_TIMEOUT", 2);

    LOG("DEBUG", "db connect " << host << ":" << port << "/" << db);

    std::ostringstream ci;
    ci << "host=" << host
       << " port=" << port
       << " user=" << user
       << " password=" << pass
       << " dbname=" << db
       << " connect_timeout=" << cto;

    PGconn* conn = PQconnectdb(ci.str().c_str());
    if (PQstatus(conn) != CONNECTION_OK) {
        LOG("ERROR", "pg connect: " << PQerrorMessage(conn));
        PQfinish(conn);
        return out;
    }

    struct PGConnGuard {
        PGconn* conn;
        PGConnGuard(PGconn* c) : conn(c) {}
        ~PGConnGuard() { if (conn) PQfinish(conn); }
    } conn_guard(conn);

    const char* sql =
        "SELECT "
        "COALESCE(t.name,''), "
        "COALESCE(t.bus_code,''), "
        "COALESCE(u.email,''), "
        "to_char(t.travel_date,'YYYY-MM-DD'), "
        "COALESCE(t.seats,1), "
        "COALESCE(t.start_node,0), "
        "COALESCE(t.end_node,0), "
        "COALESCE(t.total_cents,0) "
        "FROM public.tickets t "
        "LEFT JOIN public.users u ON u.id = t.user_id "
        "ORDER BY t.updated_at DESC "
        "LIMIT 500";

    PGresult* res = PQexec(conn, sql);
    if (PQresultStatus(res) != PGRES_TUPLES_OK) {
        LOG("ERROR", "pg query: " << PQerrorMessage(conn));
        PQclear(res);
        return out;
    }

    struct PGResultGuard {
        PGresult* res;
        PGResultGuard(PGresult* r) : res(r) {}
        ~PGResultGuard() { if (res) PQclear(res); }
    } res_guard(res);

    int rows = PQntuples(res);
    for (int i = 0; i < rows; ++i) {
        TicketRow tr;

        int name_len = PQgetlength(res, i, 0);
        const char* name_val = PQgetvalue(res, i, 0);
        tr.name.assign(name_val ? name_val : "", name_len);

        int bus_len = PQgetlength(res, i, 1);
        const char* bus_val = PQgetvalue(res, i, 1);
        tr.bus_code.assign(bus_val ? bus_val : "", bus_len);

        int email_len = PQgetlength(res, i, 2);
        const char* email_val = PQgetvalue(res, i, 2);
        tr.user_email.assign(email_val ? email_val : "", email_len);

        int date_len = PQgetlength(res, i, 3);
        const char* date_val = PQgetvalue(res, i, 3);
        tr.travel_date.assign(date_val ? date_val : "", date_len);
        
        tr.seats = safe_stoi(PQgetvalue(res, i, 4), 1);
        tr.start_node = safe_stoi(PQgetvalue(res, i, 5), 0);
        tr.end_node = safe_stoi(PQgetvalue(res, i, 6), 0);
        tr.total_cents = safe_stoll(PQgetvalue(res, i, 7), 0);

        out.push_back(std::move(tr));
    }

    return out;
}

static std::vector<TicketRow> mock_rows() {
    std::vector<TicketRow> v;
    
    std::uniform_int_distribution<> start_dist(0, 11);
    std::uniform_int_distribution<> end_dist(30, 89);
    
    for (int i = 0; i < 5; i++) {
        TicketRow tr;
        tr.name = "MOCK-" + std::to_string(i + 1);
        tr.bus_code = std::to_string(i);
        tr.user_email = "—";
        tr.travel_date = "2025-01-01";
        tr.seats = 1;
        tr.start_node = start_dist(g_gen);  
        tr.end_node = end_dist(g_gen);     
        tr.total_cents =
            llabs(tr.end_node - tr.start_node) * 50LL * 100LL;
        v.push_back(tr);
    }
    return v;
}

static void refresh_loop(const Env env) {
    const int period = env.get_int("PULL_INTERVAL_SECONDS", 10);
    fs::path data_dir = env.get("DATA_DIR", "./data");
    fs::path latest = data_dir / "latest.json";

    LOG("INFO", "refresh every " << period << "s");

    while (true) {
        auto rows = fetch_postgres(env);
        bool using_mock = rows.empty();
        if (using_mock) rows = mock_rows();

        std::string json = to_json(rows, using_mock ? "mock" : "parser");

        {
            std::lock_guard<std::mutex> lk(g_json_mx);
            g_latest_json = json;
        }

        try {
            write_file_atomic(latest, json);
        } catch (const std::exception& e) {
            LOG("ERROR", "Failed to write latest.json: " << e.what());
        }
        
        std::this_thread::sleep_for(std::chrono::seconds(period));
    }
}

static void load_bootstrap_latest(const Env& env) {
    fs::path latest = fs::path(env.get("DATA_DIR", "./data")) / "latest.json";
    if (fs::exists(latest)) {
        try {
            std::ifstream in(latest, std::ios::binary);
            if (!in) {
                throw std::runtime_error("Failed to open file");
            }
            
            std::ostringstream ss;
            ss << in.rdbuf();
            if (!in) {
                throw std::runtime_error("Failed to read file");
            }
            
            std::lock_guard<std::mutex> lk(g_json_mx);
            g_latest_json = ss.str();
        } catch (const std::exception& e) {
            LOG("ERROR", "bootstrap load failed: " << e.what());
        }
    } else {
        auto rows = mock_rows();
        std::lock_guard<std::mutex> lk(g_json_mx);
        g_latest_json = to_json(rows, "mock");
    }
}

static void serve(const Env& env) {
    std::string host = env.get("LISTEN_HOST", "127.0.0.1");
    int port = env.get_int("LISTEN_PORT", 9099);

    int srv = socket(AF_INET, SOCK_STREAM, 0);
    if (srv < 0) {
        perror("socket");
        return;
    }

    struct SocketGuard {
        int fd;
        SocketGuard(int f) : fd(f) {}
        ~SocketGuard() { if (fd >= 0) close(fd); }
    } srv_guard(srv);

    int opt = 1;
    if (setsockopt(srv, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt)) < 0) {
        perror("setsockopt");
        return;
    }

    sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(port);
    if (inet_pton(AF_INET, host.c_str(), &addr.sin_addr) <= 0) {
        LOG("ERROR", "Invalid listen address: " << host);
        return;
    }

    if (bind(srv, (sockaddr*)&addr, sizeof(addr)) < 0) {
        perror("bind");
        return;
    }
    if (listen(srv, 16) < 0) {
        perror("listen");
        return;
    }

    LOG("INFO", "Listening on " << host << ":" << port);

    while (true) {
        sockaddr_in cli{};
        socklen_t cl = sizeof(cli);
        int client_fd = accept(srv, (sockaddr*)&cli, &cl);
        if (client_fd < 0) {
            perror("accept");
            continue;
        }

        struct ClientSocketGuard {
            int fd;
            ClientSocketGuard(int f) : fd(f) {}
            ~ClientSocketGuard() { if (fd >= 0) close(fd); }
        } client_guard(client_fd);

        std::string request;
        request.reserve(4096);

        char ch;
        int total = 0;
        while (total < 4096) {
            ssize_t r = recv(client_fd, &ch, 1, 0);
            if (r <= 0) break;
            request.push_back(ch);
            total++;
            if (ch == '\n') break;
        }

        std::string json;
        {
            std::lock_guard<std::mutex> lk(g_json_mx);
            json = g_latest_json;
        }
        if (json.empty()) {
            json = to_json(mock_rows(), "mock");
        }

        json.push_back('\n');
        
        ssize_t bytes_sent = send(client_fd, json.data(), json.size(), 0);
        if (bytes_sent != static_cast<ssize_t>(json.size())) {
            LOG("WARN", "Partial send to client");
        }
    }
}

int main() {
    Env env;
    if (fs::exists(".env")) env.load_file(".env");
    load_bootstrap_latest(env);
    std::thread t(refresh_loop, env);
    t.detach();
    serve(env);
    return 0;
}
```

# Flag

```
HTB{7H3_8U773RFLY_3FF3C7_W0RK5_3V3RYWH3R3!!}
```
