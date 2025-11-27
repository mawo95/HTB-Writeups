# Analyse the binary

``` int32_t main(int32_t argc, char** argv, char** envp)
void* fsbase
int64_t rax = *(fsbase + 0x28)
int32_t var_18 = 0
int32_t var_14 = 0
setup()
puts("Deep beneath Kageno, a forgotten vault stirs.")
puts("A rusted mechanism hums faintly, waiting for a code long lost.")
int64_t rax_1
rax_1.b = 0
printf("\nEnter code> ")
fflush(*stdout)
int64_t rax_2
rax_2.b = 0
__isoc23_scanf(&data_402089, &var_14)
int64_t rax_3
rax_3.b = 0
printf("Etched letters start to appear... ")
fflush(*stdout)
sleep(1)
check_pin(var_14)
int32_t var_1c = 0
while (var_1c u< 0xc)
    putchar((*"Invalid code")[var_1c])
    fflush(*stdout)
    usleep(0x186a0)
    var_1c += 1
putchar(0xa) if (*(fsbase + 0x28) == rax)
    return 0
__stack_chk_fail()
noreturn          int64_t setup()
void* fsbase
int64_t rax = *(fsbase + 0x28)
struct sigaction act
act.__sigaction_handler.sa_handler = handler
var_a0
sigemptyset(&var_a0)
act.sa_flags = 4
if (sigaction(8, &act, nullptr) == 0xffffffff)
    perror("sigaction")
    exit(1)
    noreturn
int64_t result = *(fsbase + 0x28)
if (result == rax)
    return result
__stack_chk_fail()
noreturn     void handler() __noreturn
int32_t rdi
int32_t var_c = rdi
int64_t rsi
int64_t var_18 = rsi
uint64_t rdx
uint64_t var_20 = rdx
char var_21 = 0x41
for (int32_t i = 0x2b; i s>= 0; i -= 1)
    int64_t i_1 = i
    rdx.w = *("=Z-Y]Y" + (i_1 << 1)) ^ 0x4d4c
    *("=Z-Y]Y" + (i_1 << 1)) = rdx.w
    rdx.w = (rol.d(*("=Z-Y]Y" + (i << 1)), 2)).w
    *("=Z-Y]Y" + (i << 1)) = rdx.w
    int64_t i_2 = i
    rdx.w = *("=Z-Y]Y" + (i_2 << 1)) ^ 0x4944
    *("=Z-Y]Y" + (i_2 << 1)) = rdx.w
    rdx.w = (ror.d(*("=Z-Y]Y" + (i << 1)), 5)).w
    *("=Z-Y]Y" + (i << 1)) = rdx.w
    int64_t i_3 = i
    rdx.w = *("=Z-Y]Y" + (i_3 << 1)) - var_21
    *("=Z-Y]Y" + (i_3 << 1)) = rdx.w
    int64_t i_4 = i
    rdx.w = (*("=Z-Y]Y" + (i_4 << 1))).b
    *("=Z-Y]Y" + (i_4 << 1)) = rdx.w
    var_21 = (*("=Z-Y]Y" + (i << 1))).b
int32_t var_2c = 0
while (var_2c u< 0x2c)
    putchar(*("=Z-Y]Y" + (var_2c << 1)))
    fflush(*stdout)
    usleep(0x186a0)
    var_2c += 1 putchar(0xa)
_exit(0)
noreturn   int32_t check_pin(int32_t arg1)
int32_t result = (divs.dp.q(arg1 + 0x154f641, arg1 - 0x4149 + arg1 + 0xac979988 + 1)).d
calculated = result
return result```

# Reading the 88-byte block from the binary

```HTB{s1gN4l_H4ndL3r$-t0_w1n?}```
