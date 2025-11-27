**Code**
```
import bisect
n = int(input())
a = list(map(int, input().split()))
d = []
for x in a:
    i = bisect.bisect_left(d, x)
    if i == len(d):
        d.append(x)
    else:
        d[i] = x
print(len(d))

```

**Flag**
```
HTB{LIS_0f_th3_f1v3}
```
