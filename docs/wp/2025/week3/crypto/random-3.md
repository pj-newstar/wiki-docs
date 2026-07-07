---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 随机数之旅 3

<Container type='info'>

本题考查线性代数。
</Container>

FLAG 的每个字符取其 ASCII 码得到明文向量 $x$，注意这是 42 维的。
随机生成 $41\times 42$ 的模 $p$ 矩阵 $A$，计算 $b=Ax$
题目输出了 $A,b,p$，目标是恢复 $x$。

根据题目关系有：
$$A_{41\times 42}x_{42\times1}=b_{41\times1}$$
根据线代知识，这是欠定方程，它的解构成一个解空间：
$$x=x_p+tv,t\in Z_p$$
其中 $x_p$ 是特解，$v$ 是 $A$ 的右核空间的基。

用 Sage 写就是：

```python
A=matrix(Zmod(p),A)
b=vector(b)

xp=A.solve_right(b)   #特解

ArK=A.right_kernel().basis()  # .basis()返回一个向量空间的基（组），
v0=ArK[0]   #右核的第一个基

```

可以通过 `rank()` 函数查看矩阵的秩。容易得到 $A$ 的秩是 $41$，因此右核只有 1 维，即只有一个基。

方程解有多个，但 FLAG 只有一个。注意到 FLAG 中有 6 个字符已知，可以任选一个已知字符计算参数：

```python
t=(ord("l")-xp[1])*pow(v0[1],-1,p)%p
x=xp+t*v0
mes=list(x)

FLAG="".join(chr(mes[i]) for i in range(42))
print(FLAG)
```
