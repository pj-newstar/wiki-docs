---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# FHE: 0 and 1

<Container type='info'>

本题考查同态加密的分析。
</Container>

$$pk_i=r_ip+s_i$$

其中 $pk$ 是 `public_key` 的值，$r_i$ 是一个随机数，$s_i$ 是一个很小的数。

显然可以把 $s_i$ 爆破然后求最大公约数。

这样会得到 $p$ 的倍数，而先前给定了 $p$ 是 $128$ 位，除以几个小因子就可以得到 $p$。

$$c_i=b_i+n_i+l_i$$

其中 $b_i$ 是每个比特，$n_i$ 是一个偶数小噪音， $l_i$ 是 $p$ 倍数的大噪音。我们刚刚求出了 $p$，模 $p$ 就可以把 $l_i$ 去掉，再对 $2$ 取模就可以去掉 $n_i$。剩下的就是 $b_i$，拼起来就是我们要求的 flag 了。
