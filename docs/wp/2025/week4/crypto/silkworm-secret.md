---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 天虫的秘密

<Container type='info'>

本题考查 Padding Oracle Attack。
</Container>

题目生成 `key` 和 `iv1`，并使用 AES-CBC 模式加密 FLAG 得到 `ct`，输出 `iv1+ct` 的 Base64 编码。
`oracle` 接收一段 `iv+ct` 的 Base64 编码，然后使用 `key` 和 `iv` 解密 `ct` 得到 `pt`，返回 `pt` 是否具有正确的 padding。
选手可以无限次交互。

> [!NOTE]
> padding:
> 这里使用了 `unpad(pt,16)`，填充协议是 PKCS#7。正确的填充格式是：当明文距离分块长度还差 $T$ 个字节（$0<T<16$）时，明文末尾添加 $T$ 个 `0xT`。

CBC 解密关系如下：

CBC 的解密过程可以表示为： $P_i=D(C_i)\oplus C_{i-1}$
其中 $P_i$ 表示明文第 $i$ 块，$C_i$ 表示密文第 $i$ 块，$D(C_i)$ 表示用 `key` 解密。
整个解题过程中无法控制 $D(C_i)$，但可以控制输入的 $C_{i-1}$，从而改变 $P_i$。如果 oracle 返回 `OK`，就说明当前 $P_i$ 符合 padding。

攻击的核心就是利用 padding 校验结果：当 padding 合法时，可以据此推导出部分明文字节。

以恢复明文最后一个字节为例。修改 $C_{i-1}$ 得到 $C_{i-1}^{\prime}$，区别是只修改 $C_{i-1}[15]$。此时明文也会变为 $P_i^{\prime}=D(C_i)\oplus C_{i-1}^{\prime}$。
从 $0$ 到 $255$ 爆破最后一个字节，直到 oracle 返回 `OK`，即可确定 $P_i^{\prime}[15]=0x01$。
于是可以恢复 $D(C_i)[15]=P_i^{\prime}[15]\oplus C_{i-1}^{\prime}[15]$，随后得到 $P_i[15]=D(C_i)[15]\oplus C_{i-1}[15]$。

之后将该过程扩展到整个分块即可。已知 $D(C_i)[15]$ 后，可以计算出 $C_{i-1}^{\prime}[15]$，使得 $P_i^{\prime}[15]=0x02$（或其他目标 padding 值）。

其中 `IV` 即 $C_0$。

该类攻击脚本较为通用，可以复用常见 Padding Oracle Attack 模板完成求解。
