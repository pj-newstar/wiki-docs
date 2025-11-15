---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 小跳蛙

<Container type='info'>

本题考查 GCD 及简单的数论知识。
</Container>

题目脚本已经告诉我们了，每次给出一个坐标 `(x,y)`，我们需要返回最后小青蛙会停留的坐标。

1. 你可以**直接复用题目脚本的验证部分**，即 `while a!=b:` 那一块。
2. 感觉直接复用太 low 了？没事你还可以学习发现：小跳蛙跳跃的过程就是在模拟辗转相除法<span data-desc>（欧几里得算法）</span>求两个数的最大公约数！这样就好办了，我们**找一个库去使用 `gcd()` 函数**就行了。
