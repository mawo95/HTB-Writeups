# Solution

```py
import sys
sys.setrecursionlimit(3000000)
from bisect import bisect_left

data = sys.stdin.read().strip().split()
it = iter(data)
n = int(next(it))
R = [0] + [int(next(it)) for _ in range(n)]
adj = [[] for _ in range(n+1)]
for _ in range(n-1):
    u = int(next(it)); v = int(next(it))
    adj[u].append(v)
    adj[v].append(u)

subsize = [0]*(n+1)
dead = [False]*(n+1)

def dfs_size(u, p):
    subsize[u] = 1
    for v in adj[u]:
        if v == p or dead[v]: continue
        dfs_size(v, u)
        subsize[u] += subsize[v]

def find_centroid(u, p, total):
    for v in adj[u]:
        if v == p or dead[v]: continue
        if subsize[v] > total//2:
            return find_centroid(v, u, total)
    return u

def collect_paths(u, p, path, out):
    # path currently is ranks from centroid -> ... -> u
    path.append(R[u])
    out.append(list(path))   # store a copy
    for v in adj[u]:
        if v == p or dead[v]: continue
        collect_paths(v, u, path, out)
    path.pop()

def lis_len_with_threshold(arr, center_val, side):
    tails = []
    if side == 'right':
        for x in arr:
            if x <= center_val: 
                continue
           
            i = bisect_left(tails, x)
            if i == len(tails):
                tails.append(x)
            else:
                tails[i] = x
    else:
        for x in reversed(arr):
            if x >= center_val:
                continue
            i = bisect_left(tails, x)
            if i == len(tails):
                tails.append(x)
            else:
                tails[i] = x
    return len(tails)

answer = 0

def decompose(start):
    global answer
    dfs_size(start, 0)
    c = find_centroid(start, 0, subsize[start])
    dead[c] = True

    center_val = R[c]

    best_right_seen = 0
    best_left_seen = 0
    answer = max(answer, 1)

    for v in adj[c]:
        if dead[v]: 
            continue
        paths = []
        collect_paths(v, c, [], paths) 
       
        max_left = 0
        max_right = 0
        for arr in paths:
           
            left_len = lis_len_with_threshold(arr, center_val, 'left')
           
            right_len = lis_len_with_threshold(arr, center_val, 'right')
            if left_len > max_left: max_left = left_len
            if right_len > max_right: max_right = right_len

            if right_len > 0:
                answer = max(answer, 1 + right_len)
            if left_len > 0:
                answer = max(answer, left_len + 1)

        if max_left > 0 and best_right_seen > 0:
            answer = max(answer, max_left + 1 + best_right_seen)
        if max_right > 0 and best_left_seen > 0:
            answer = max(answer, max_right + 1 + best_left_seen)

        if max_right > best_right_seen:
            best_right_seen = max_right
        if max_left > best_left_seen:
            best_left_seen = max_left

    for v in adj[c]:
        if not dead[v]:
            decompose(v)

answer = 0
decompose(1)
print(answer)
```

# Flag
```
HTB{bl4d3_s3qu3nc3_unbr0k3n}
```

```
