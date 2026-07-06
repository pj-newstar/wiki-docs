---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# CBC 之舞

这道题是一个基于 AES-CBC 模式的加密题目，CBC 是「密码块链接」模式。

其特点是：每个明文块在加密前会先与前一个密文块进行异或（XOR），第一个块则与初始化向量（IV）进行异或。如果密文块顺序被打乱，解密时会导致明文块错位，从而可能泄露信息。

我们已知以下数据：

```plaintext
IV1 (hex): 1e5d251ea78ef68a1282079fd028c747
IV2 (hex): 18777ae4c1a29f4c5db8ba6c5dfe72f1
m1 (hex): f560fd28ed5c5ce7d952eb44b47007e702f42dbb54540dfc78467f48933dbb01ebcf520fd3d23a211d3b4e8c06261966cb178525c25b8058ff792e0f251d3d15
c1 (hex): caf7bc1223c17f848aec854a87b8958d4c518f7287663bfae0b6a5a1e0f0eb95b50c9ea6789a7d77fda5f50d1b8a2183b40cab693ebacf32a9b59faf3b0084ff
c2 (hex): b40cab693ebacf32a9b59faf3b0084ffcaf7bc1223c17f848aec854a87b8958db50c9ea6789a7d77fda5f50d1b8a21834c518f7287663bfae0b6a5a1e0f0eb95
```

值得注意的是，`c1` 是 `c2` 的四个块经过某种置换（permutation）后得到的。

相关代码如下：

```python
perm = [1, 2, 3, 0]
random.shuffle(perm)
while any(i == perm[i] for i in range(4)):
    random.shuffle(perm)
```

观察后不难发现：

```python
c1[0] = c2[3]
c1[1] = c2[0]
c1[2] = c2[2]
c1[3] = c2[1]
```

即 `perm = [3, 0, 2, 1]`。

CBC 模式中的解密过程为：

```plaintext
P_i = Decrypt(C_i) XOR C_{i-1}
```

利用 CBC 的解密公式，可以建立 `m1` 与 `c1`、`c2` 之间的关系，从而还原 `m2` 的内容。

核心解密公式如下：

```plaintext
m2[i] = AES_Decrypt(c2[i]) XOR c2[i-1]
```

而 `AES_Decrypt(c2[i])` 实际上可以表示为 `m1[j] XOR c1[k]` 的形式，通过置换关系可以一一对应。

最终推导出四个 m2 块：

```python
m20 = xor(xor(m1[3], c1[2]), IV2)
m21 = xor(xor(IV1, m1[0]), c2[0])
m22 = xor(xor(c1[1], m1[2]), c2[2])
m23 = xor(xor(c1[0], m1[1]), c2[1])
```

完整 EXP：

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from Crypto.Util.number import*
import os
from pwn import xor

IV1 =0x1e5d251ea78ef68a1282079fd028c747
IV2 =0x18777ae4c1a29f4c5db8ba6c5dfe72f1
m1 =0xf560fd28ed5c5ce7d952eb44b47007e702f42dbb54540dfc78467f48933dbb01ebcf520fd3d23a211d3b4e8c06261966cb178525c25b8058ff792e0f251d3d15
c1 =0xcaf7bc1223c17f848aec854a87b8958d4c518f7287663bfae0b6a5a1e0f0eb95b50c9ea6789a7d77fda5f50d1b8a2183b40cab693ebacf32a9b59faf3b0084ff
c2 =0xb40cab693ebacf32a9b59faf3b0084ffcaf7bc1223c17f848aec854a87b8958db50c9ea6789a7d77fda5f50d1b8a21834c518f7287663bfae0b6a5a1e0f0eb95

IV1=long_to_bytes(IV1)
IV2=long_to_bytes(IV2)
m1=long_to_bytes(m1)
c1=long_to_bytes(c1)
c2=long_to_bytes(c2)

m20=xor(xor((m1[-16:]),c1[-32:-16]),IV2)
m21=xor(xor(IV1,m1[:16]),c2[:16])
m22=xor(xor(c1[16:32],m1[-32:-16]),c2[16:32])
m23=xor(xor(c1[:16],m1[16:32]),c2[-32:-16])
print(m20,m21,m22,m23)
```
