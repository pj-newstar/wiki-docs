---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 随机数之旅 1

<Container type='info'>

题考查 LCG 流密码。
</Container>

`flag` 转为整数 `b`，容易知道题目给出了关系式：

$$X_{i+1}=aX_i+b\;\;\pmod{p}$$

$a,p$ 已知，同时给了连续 $5$ 个 $X_i$ ，很容易计算出以下结果。

$$b=X_{i+1}-aX_{i}\;\;\pmod{p}$$

<details>
  <summary>fun fact</summary>

题目文件的名字是`random_jerni1`，`jerni`是谐音 journey.

</details>
