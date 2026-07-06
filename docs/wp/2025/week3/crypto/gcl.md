---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# GCL

<Container type='info'>

本题考查 LCG 相关分析。
</Container>

FLAG 转数字得到 `m`。
素数$p$，两随机数$a,b$，序列 $\{s_n\}$ 满足关系：$$s_{n}=as^{-1}_{n-1}+b\;\pmod{p}$$
`gift` 给了连续 10 项 `s`，我们需要求出下一项作为 key 解密 `c=m^key`

和 LCG 类似，我们肯定要先求出 $p$ 出来。

$$
\begin{align}s_{i}&=as^{-1}_{i-1}+b\;\pmod{p}\\
s_is_{i-1}&=a+bs_{i-1}\pmod p\;\;(消掉模逆)\\
s_{i+1}s_i&=a+bs_i\pmod p\;\;(再写一项)\\
s_{i+1}s_i-s_is_{i-1}&=b(s_i - s_{i-1})\pmod p\;\;(消掉 a)\\
s_i(s_{i+1} - s_{i-1})&=b(s_i - s_{i-1})\pmod p\;\;(整理一下)\\
s_{i+1} * (s_{i+2} - s_i) &= b(s_{i+1} - s_i) \pmod p\;\;(再写一项)\\
\end{align}
$$

交叉相乘把 $b$ 消掉：

$$s_i(s_{i+1} - s_{i-1})(s_{i+1} - s_i) - s_{i+1}(s_{i+2} - s_i)(s_i - s_{i-1}) = 0 \pmod p$$

求 GCD 可以求出 $p$ (也可能是倍数)，之后也可以求出 $b,a$
