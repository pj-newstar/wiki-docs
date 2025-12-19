---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 唯一表示

<Container type='info'>

本题考查 CRT.
</Container>

这里简单介绍一个函数：

`n, k = crt(primes, remainders)` 这是 [sympy 库](https://docs.sympy.org/latest/index.html)的一个用于求解线性同余方程组的函数。

> [!TIP] （一元）线性同余方程组
> （其中 $n_1,\cdots,n_k$ 两两互质）
> $$\begin{cases}x&\equiv a_1({\mathrm{mod}}\;n_1)\\x&\equiv a_2({\mathrm{mod}}\;n_2)\\&\vdots\\x&\equiv a_k({\mathrm{mod}}\;n_k)&\end{cases}$$

`primes` 就是要输入数组 `[n_1,...,n_k]` ，`remainders` 就是要输入数组 `[a_1,...,a_k]`。

`n` 是该函数返回的最小的 `x`<span data-desc>（显然方程组有一组解，这些解在模的意义下等同）</span>,`k` 就是模。

题目要求的 `flag` 转换成了整数，然后使用了 `fun()` 函数处理，得到了一个数组。容易知道该数组就是上面讲的 `remainders`，而 `primes` 已经告诉了怎么生成的。**直接复用 `crt()` 就可以求解**，再把数字转换为字节即可。

<details>
  <summary>fun fact</summary>

题目文件的名字是 `unik_repri`，其中 `unik` 是谐音 unique ，`repri` 是取了 representation 的开头几个音节。

</details>
