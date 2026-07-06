---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 不给你看喵

<Container type='info'>

本题考查 01 背包问题与零知识证明的结合。
</Container>

题目脚本如下：

```python
import random
import math
from secret import FLAG

p=225791639467198034995070527100776477487
g=3
h=5

def round(n):
  a=[random.randint(1,p-1) for _ in range(n)]
  x=[random.randint(0,1) for _ in range(n)]
  t=sum(a[i]*x[i] for i in range(n))
  print(a)
  print(t)    # 此处可以通过格规约恢复 x。

  C=[]     # C 是选手输入的数组，用于证明其知道 x 的每一位。
  for _ in range(n):
    bit=int(input("Every bit: "))
    C.append(bit)

  s=[random.randint(1,p-1) for _ in range(n)]
  print(s)

  S=int(input(">"))
  R=int(input(">"))

  assert R>S
  assert S==sum(s[i]*x[i] for i in range(n)) # S 必须是 s 的子集和。
  assert all(C[i]>1 for i in range(n))  # C 的每个元素都需要大于 1。
  assert math.prod(pow(C[i],s[i],p) for i in range(n))%p==pow(g,S,p)*pow(h,R,p)%p

round(16)

print(FLAG)
```

核心约束是最后一个 `assert` 语句：

$$\prod_{i}C[i]^{s[i]} =g^Sh^R\pmod{p}$$

将其改写为：

$$\prod_{i}C[i]^{s[i]} =g^{\sum_i s[i]x[i]}h^R=(\prod_ig^{s[i]x[i]})h^R \pmod{p}$$

`R` 由选手提交，因此可以令：

$$R=\sum_i s[i]r[i]$$

其中 `r` 是自行选择的数组。代入后可得：

$$\prod_{i}C[i]^{s[i]} =g^{\sum_i s[i]x[i]}h^{\sum_is[i]r[i]}=\prod_ig^{s[i]x[i]}h^{s[i]r[i]}\pmod{p}$$

因此构造：

$$C[i]=g^{x[i]}h^{r[i]}$$

同时需要满足 `R > S`，即保证：

$$\sum_i r[i]>\sum_i x[i]$$

至此可以通过格规约恢复 `x`，再按上述方式构造 `C`、`S`、`R` 完成交互。

参考脚本如下：

```python
a=
t=

p=
n=16

Ge=matrix(ZZ,n+1,n+1)
for i in range(n):
    Ge[i,i]=1
    Ge[i,-1]=a[i]
Ge[-1,-1]=-t
Ge=Ge.LLL()
x=list(Ge[0])[:-1]
print(x)

import random
g=3
h=5
r=[1]*16
C=[g^x[i]*h^r[i] for i in range(n)]
print(C)
s=
S=sum(s[i]*x[i] for i in range(n))
R=sum(s[i]*r[i] for i in range(n))
print(S,R)
```
