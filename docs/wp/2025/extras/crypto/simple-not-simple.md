---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 简约但不简单

```python
import uuid
from random import getrandbits as grb

FLAG="FLAG{"+str(uuid.uuid4())+"}"
a,b=[grb(32) for _ in range(2)]
pd={}

def f(s,j):
  n=len(s)
  return sum(ord(s[i])*j**(n-1-i) for i in range(n))

for _ in range(3):
  x=grb(32)
  pd[x]=a*f(FLAG,x)+b

print("pd=",pd)
```

EXP：

```python
pd={}
t=list(pd.keys())
ft=[pd[i] for i in t]

Ge=matrix(ZZ,44,42+n)
for i in range(42):
    Ge[i,i]=1
    for j in range(n):
        Ge[i,42+j]=t[j]**(41-i)
        Ge[-2,42+j]=-1
        Ge[-1,42+j]=-ft[j]
Ge[42,42]=1
Ge=Ge.LLL()
#print(Ge)
#print(Ge[0])
#print(Ge[1])
#print(Ge[2])
print(Ge[3])

x=list(Ge[3])
assert x[0]!=0
from math import gcd
aa=gcd(x[0],x[1],x[2])
print(aa)
x=[x[i]//aa for i in range(42)]
print(x)
print("".join(chr(int(x[i])) for i in range(0,41)))
```

要点：题目给出的是 `a*f(flag,x)+b`，本质上与给出 `f(flag,x)` 没有区别。常数项会与 $b$ 共同作用，而 $a$ 可以通过 GCD 消去。
