---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# DLP

这是 week2DLP_1 的升级版，这里使用到了几个 sagemath 中的函数，其他基础知识在 DLP_1 和 week1 的题目中已有涉及到，解题脚本如下：

```python
from Crypto.Util.number import long_to_bytes
N =
y =
fac = factor(N)
primes = []
for p, e in fac:
    for _ in range(e):
        primes.append(Integer(p))

residues = []
moduli = []

for idx, p in enumerate(primes, start=1):
    p = Integer(p)
    # 找一个原根 g
    g = primitive_root(p)   # Sage 内置函数
    yi = Integer(y % p)
    d = discrete_log(Mod(yi, p), Mod(g, p))
    residues.append(Integer(d))
    moduli.append(Integer(p - 1))

# 使用中国剩余定理合并所有同余
x_recovered = crt(residues, moduli)
print("Recovered integer x (mod product of moduli) =", x_recovered)

# 4) 把整数转回 bytes -> str
flag_bytes = long_to_bytes(int(x_recovered))
print(flag_bytes)
```
