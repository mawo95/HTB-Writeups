# Analyse the binary

```
int32_t main(int32_t argc, char** argv, char** envp)
void* fsbase
int64_t rax = *(fsbase + 0x28)
puts("[Core] Resonance mismatch.")
puts("[Sentinel] Purpose altered. Cannot authenticate echo.")
printf("Enter resonance pattern:\n> ")
char buf[0x88]
int32_t result
if (fgets(&buf, 0x80, __bss_start) != 0)
    stone_shift(&buf)
    hum_resonance(&buf)
    result = 0
else
    puts("No input. Echo faded.")
    result = 1
*(fsbase + 0x28)
if (rax == *(fsbase + 0x28))
    return result
__stack_chk_fail()
noreturn  int64_t purpose_overwrite(int32_t arg1)
if (arg1 == 0)
    return puts("Pattern rejected. Echo lost.")
puts("[Core] Resonance aligned.")
puts("[Sentinel] Echo stabilized. Purpose understood.")
return puts("Pattern accepted.") int64_t hum_resonance(char* arg1)
if (verify_pattern() != 0)
    puts("[Sentinel] Pattern misread. Echo unstable...")
return purpose_overwrite(forging_cycle_realign(arg1))  int64_t deprecated_core_shift(char* arg1)
void* fsbase
int64_t rax = *(fsbase + 0x28)
char var_38[0x18]
memset(&var_38, 0, 0x19)
for (int64_t i = 0; i u<= 0x17; i += 1)
    var_38[i] = (*"xdrKB")[i] ^ 0x30
char var_20 = 0
int32_t var_4c = 0
for (int64_t i_1 = 0; i_1 u<= 0x17; i_1 += 1)
    var_4c += (i_1.d * 3) ^ (*"xdrKB")[i_1]
int32_t var_4c_1 = var_4c ^ 0x5a
int64_t result
if (strcmp(arg1, &var_38) != 0)
    result = 0
else
    result = 1
*(fsbase + 0x28)
if (rax == *(fsbase + 0x28))
    return result
__stack_chk_fail()
noreturn   uint64_t verify_pattern() __pure
int64_t rdi
int64_t var_20 = rdi
int32_t var_10 = 0
for (int32_t i = 0; i s<= 4; i += 1)
    var_10 += i * 3
int32_t rax
rax.b = var_10 == 0x2a
return rax.b
  int64_t forging_cycle_realign(char* arg1)
int32_t var_10 = 0
for (int32_t i = 0; i s<= 9; i += 1)
    var_10 ^= (i * 7) ^ 0x2a
return deprecated_core_shift(arg1)  void stone_shift(char* arg1)
if (arg1 == 0)
    return 
uint64_t var_10_1 = strlen(arg1)
if (var_10_1 != 0 && arg1[var_10_1 - 1] == 0xa)
    arg1[var_10_1 - 1] = 0 und jetzt gib die flag aus iron
```

# Reading bytes
```HTB{r3wr1tt3n_r3s0nanc3}```
