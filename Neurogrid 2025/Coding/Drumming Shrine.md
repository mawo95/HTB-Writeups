**Code**

```
n = int(input())
a = list(map(int, input().split()))
pi = [0] * n
j = 0
for i in range(1, n):
    while j > 0 and a[i] != a[j]:
        j = pi[j - 1]
    if a[i] == a[j]:
        j += 1
    pi[i] = j
p = n - pi[-1]
print("YES" if p < n and n % p == 0 else "NO")
```

**Flag**
```
HTB{3t3rn4l_sh1nju_p4tt3rn}
```
