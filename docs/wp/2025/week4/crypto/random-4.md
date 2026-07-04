---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 随机数之旅 4

<Container type='info'>

本题考查线性代数。
</Container>

FLAG 被切片为 14 块，每块转为数字后得到秘密向量 $c$。
$x$ 初始为 14 个随机数字，然后按如下关系扩展 $x$ 序列：

$$x_k = (c_0*x_{k-14} + c_1*x_{k-13} + ... + c_{13}*x_{k-1}) \pmod{p}$$

题目给出了 $p$ 和 $x$ 序列的最后 28 位。

利用这 28 位可以根据递推关系列出 14 个关于 $c_i$ 的 14 元方程，求解即可。

```python
p=
x=

n=14
A=matrix(Zmod(p),n,n)
for i in range(n):
    for j in range(n):
        A[i,j]=x[i+j]
b=x[-14:]
c=A.solve_right(b)

m=b"".join(long_to_bytes(int(c[i])) for i in range(n))

```
