---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 共轭迷宫

### 题目背景知识

1. DH 密钥交换协议

DH（Diffie-Hellman）协议是一种密钥交换算法，允许两个通信方在不安全的信道上建立一个共享密钥。基本思想是：Alice 和 Bob 各自生成私钥 $a$ 和 $b$，计算公钥 $P_A = g^a \bmod p$、$P_B = g^b \bmod p$。交换公钥后，各自计算共享密钥 $K = \text{对方公钥}^{\text{自己的私钥}} \bmod p$。由于 $g^{ab} = g^{ba}$，双方得到相同的共享密钥。

2. 四元数

四元数是复数的扩展，形式为 $q = w + xi + yj + zk$。

- $w$ 是实部，$x, y, z$ 是三个虚部。

- $i^2 = j^2 = k^2 = ijk = -1$。

- 四元数乘法不满足交换律：$q_1 \times q_2 \ne q_2 \times q_1$。

- 共轭四元数：$\operatorname{conj}(q) = w - xi - yj - zk$。

- 逆四元数：$q^{-1} = \operatorname{conj}(q) / ||q||^2$。

### 题目分析

本题将 DH 协议推广到四元数领域：生成元 $g$ 由 FLAG 编码而成，Alice 和 Bob 使用四元数私钥 $a, b$。

公钥计算：$P_A = a \times g \times a^{-1}$，$P_B = b \times g \times b^{-1}$。

共享密钥：$K = a \times P_B \times a^{-1} = b \times P_A \times b^{-1}$。

关键观察：由于共轭运算的特性，共享密钥 $K$ 实际上等于 $b \times a \times g \times a^{-1} \times b^{-1}$。

题目给出了归一化后的共享密钥 $K$ 和原始 $g$ 的模平方 `norm_squared`，由此可以得出原始的 $K$。

```python
def recover_from_norm_squared(normalized_quat, norm_squared):
    getcontext().prec = 50
    norm = Decimal(norm_squared).sqrt()
    w_recovered = norm * Decimal(normalized_quat.w)
    x_recovered = norm * Decimal(normalized_quat.x)
    y_recovered = norm * Decimal(normalized_quat.y)
    z_recovered = norm * Decimal(normalized_quat.z)
    return Quaternion(w_recovered, x_recovered, y_recovered, z_recovered)
```

在四元数 DH 中：

$K = a \times (b \times g \times b^{-1}) \times a^{-1} = (a \times b) \times g \times (b^{-1} \times a^{-1})$。

由于 $K$ 已知，我们需要找到 $g$。

关键技巧：题目中私钥 $a, b$ 是特殊构造的（45° 和 60° 旋转），它们的乘积具有可交换性，$a$ 与 $a^{-1}$ 相消，$b$ 与 $b^{-1}$ 相消，使得 $K = g$。

注意得到的 $K$ 的四个分量，需要使用题目提供的后六位数据进行微调（这是在转换中出现的数据的微小差异 1）。

### 完整解题代码

```python
import math

from Crypto.Util.number import long_to_bytes
from torchvision.transforms.v2.functional import normalize
from decimal import Decimal, getcontext

def recover_from_norm_squared(normalized_quat, norm_squared):
    getcontext().prec = 50

    norm = Decimal(norm_squared).sqrt()
    w_norm = Decimal(normalized_quat.w)
    x_norm = Decimal(normalized_quat.x)
    y_norm = Decimal(normalized_quat.y)
    z_norm = Decimal(normalized_quat.z)

    w_recovered = norm * w_norm
    x_recovered = norm * x_norm
    y_recovered = norm * y_norm
    z_recovered = norm * z_norm

    return Quaternion(w_recovered, x_recovered, y_recovered, z_recovered)

import numpy as np
from math import sqrt
import hashlib

import numpy as np
from math import sqrt
import hashlib
from decimal import Decimal, getcontext

class Quaternion:
    def __init__(self, w, x, y, z):
        self.w = w
        self.x = x
        self.y = y
        self.z = z

    def __mul__(self, other):
        w1, x1, y1, z1 = self.w, self.x, self.y, self.z
        w2, x2, y2, z2 = other.w, other.x, other.y, other.z
        w = w1 * w2 - x1 * x2 - y1 * y2 - z1 * z2
        x = w1 * x2 + x1 * w2 + y1 * z2 - z1 * y2
        y = w1 * y2 - x1 * z2 + y1 * w2 + z1 * x2
        z = w1 * z2 + x1 * y2 - y1 * x2 + z1 * w2
        return Quaternion(w, x, y, z)

    def inv(self):
        norm_sq = self.w**2 + self.x**2 + self.y**2 + self.z**2
        if abs(norm_sq) < 1e-10:
            raise ValueError("Cannot invert quaternion with zero norm")
        return Quaternion(self.w/norm_sq, -self.x/norm_sq, -self.y/norm_sq, -self.z/norm_sq)

    def conjugate(self):
        return Quaternion(self.w, -self.x, -self.y, -self.z)

    def norm(self):
        getcontext().prec = 50
        w = Decimal(self.w)
        x = Decimal(self.x)
        y = Decimal(self.y)
        z = Decimal(self.z)
        norm_sq = w * w + x * x + y * y + z * z
        n = norm_sq.sqrt()

        return Quaternion(w/n, x/n, y/n, z/n),norm_sq,n

    def __str__(self):
        return f"{self.w}+{self.x}i+{self.y}j+{self.z}k"

    def __eq__(self, other):
        return (abs(self.w - other.w) < 1e-10 and
                abs(self.x - other.x) < 1e-10 and
                abs(self.y - other.y) < 1e-10 and
                abs(self.z - other.z) < 1e-10)

norm_squared=15960922284361974605582033637987025644912788
normalized=Quaternion( 0.47292225874042030768,0.44018598307489329914,0.54174915328248053438,0.53766968708489053908)
recovered = recover_from_norm_squared(normalized, norm_squared)
print(f"恢复的原始: {recovered}")
w,x,y,z=1889377532526941271603,1758592434980910292847,2164348705437511939167,2148050779856765994109
print(long_to_bytes(w),long_to_bytes(x))
print(long_to_bytes(y),long_to_bytes(z))
```
