---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 被泄露的素数

这是一道结合 RSA 加密和 Coppersmith 攻击的题目。

加密脚本中关于 `p` 的处理如下：

```python
p_bits = int(p).bit_length()  # p 的位数，应该是 1024
high_bit_count = int(p_bits * 2/3)  # 取 p 的高 2/3 位，约 682 位
p_high = p >> (p_bits - high_bit_count)  # 右移得到高 682 位

mask = (1 << (high_bit_count - '?')) - 1  # 这里 '?' 是一个未知数
p_high_masked = p_high & mask  # 用掩码隐藏一部分高位
```

通过位数估计可以发现，`?` 掩码隐藏的高位数量不多，仍在可爆破范围内。

对于泄露高位的 `p`，主要使用 Coppersmith 方法求解。

Coppersmith 方法是用来求解模方程小根的一种算法。

在这里，我们已知 p 的高位，未知低位，可以设：

```python
p = p0 + x
```

其中 p0 是已知的高位部分，x 是未知的低位部分（小于 2^342）。

因为 n = p \* q，所以：

```plaintext
f(x) = p0 + x 是 n 的一个小因子
```

Coppersmith 可以在模 n 下找到这样的小根 x。

在 SageMath 的 `small_roots()` 方法中，主要有两个重要参数：

1. `X`：根的上界

```python
roots = f.small_roots(X=2^342, beta=0.44)
```

`X` 表示预期根的最大绝对值。

在我们的题目中：p 是 1024 位，已知 p 的高 682 位，所以未知的低位部分有：1024 \- 682 = 342 位，因此 x < 2^342，所以设置 X = 2^342。

设置技巧：如果 X 设得太小，可能找不到真正的根，如果 X 设得太大，计算会变慢甚至失败，一般根据未知的位数来设置：X = 2^\(未知位数\)。

2. `beta`：因子的相关大小

`beta` 表示所求因子与模数的比例关系。在 RSA 中，`n = p × q`。通常 `p` 和 `q` 大小相近，约为 `sqrt(n)`，因此 `p ≈ n^0.5`，对应 `beta ≈ 0.5`。

为什么这里用 0\.44 而不是 0\.5？

理论上，如果 `p` 和 `q` 完全等长，可以使用 `beta = 0.5`。但实际题目中二者可能不完全相等，因此通常将该值略微调低，例如 `0.48`、`0.45` 或 `0.44`。如果 `0.5` 无法得到根，可以逐步尝试更小的经验值。

完整 EXP：

```python
from Crypto.Util.number import *
from sage.all import*

## 读取公钥文件
def read_public_key(filename):
    with open(filename, 'r') as f:
        content = f.read().strip()

    n = None
    e = None

    # 处理不同的公钥格式
    if 'n = ' in content and 'e = ' in content:
        # 格式: n = 0x... e = 0x...
        n_str = content.split('n = ')[1].split()[0]
        e_str = content.split('e = ')[1].split()[0]

        # 处理十六进制或十进制
        if n_str.startswith('0x'):
            n = int(n_str, 16)
        else:
            n = int(n_str)

        if e_str.startswith('0x'):
            e = int(e_str, 16)
        else:
            e = int(e_str)

    elif '-----BEGIN PUBLIC key-----' in content:
        # PEM 格式的公钥（需要解析）
        from Crypto.PublicKey import RSA
        key = RSA.import_key(content)
        n = key.n
        e = key.e
    else:
        # 尝试直接解析为数字
        lines = content.split('\n')
        for line in lines:
            if '=' in line:
                key, value = line.split('=', 1)
                key = key.strip()
                value = value.strip()
                if key == 'n':
                    if value.startswith('0x'):
                        n = int(value, 16)
                    else:
                        n = int(value)
                elif key == 'e':
                    if value.startswith('0x'):
                        e = int(value, 16)
                    else:
                        e = int(value)

    return n, e

## 读取 partial_p.txt
def read_partial_p(filename):
    with open(filename, 'r') as f:
        content = f.read().strip()

    # 处理???开头的格式
    if content.startswith('???'):
        hex_str = content[3:]  # 去掉前 3 个字符"???"
        # 去除可能存在的空格或其他分隔符
        hex_str = hex_str.replace(' ', '').replace(':', '')
        base_val = int(hex_str, 16)
        return base_val, hex_str
    else:
        # 如果没有???，直接解析为数字
        if content.startswith('0x'):
            return int(content, 16), content[2:]
        else:
            return int(content), content

## 读取密文文件
def read_ciphertext(filename):
    with open(filename, 'rb') as f:
        ciphertext_bytes = f.read()

    # 尝试判断是原始字节还是编码后的
    try:
        # 如果是十六进制编码的
        if len(ciphertext_bytes) > 2 and ciphertext_bytes[:2] == b'0x':
            return int(ciphertext_bytes[2:].decode(), 16)
        # 如果是 Base64 编码的
        elif b'BEGIN' in ciphertext_bytes or b'BASE64' in ciphertext_bytes:
            from base64 import b64decode
            return bytes_to_long(b64decode(ciphertext_bytes))
        else:
            # 直接作为字节处理
            return bytes_to_long(ciphertext_bytes)
    except:
        # 如果解析失败，直接返回字节的整数形式
        return bytes_to_long(ciphertext_bytes)

## 主读取函数
def read_challenge_files():
    try:
        # 读取公钥
        n, e = read_public_key('public_key.pem')
        print(f"读取到 n: {n}")
        print(f"读取到 e: {e}")
        print(f"n 的位数: {n.bit_length()}")

        # 读取部分 p
        base_val, hex_str = read_partial_p('partial_p.txt')
        print(f"读取到部分 p: ???{hex_str}")
        print(f"基础值: {hex(base_val)}")
        print(f"基础值位数: {base_val.bit_length()}")

        # 读取密文
        c = read_ciphertext('ciphertext.bin')
        print(f"读取到密文: {hex(c)}")
        print(f"密文长度: {c.bit_length()} 位")

        return n, e, base_val, c

    except FileNotFoundError as e:
        print(f"文件未找到: {e}")
        return None, None, None, None
    except Exception as e:
        print(f"读取文件时出错: {e}")
        return None, None, None, None

n, e, base_val, c = read_challenge_files()
def solve():
    n,e,base_val,c=read_challenge_files()
    for msb in range(0,8):
        full_high=(msb<<(682-3))| base_val
        p0=full_high<<342
        print(p0.bit_length())

        PR.<x>=PolynomialRing(Zmod(n))
        f=p0+x
        roots=f.small_roots(X=2^342,beta=0.44)
        print(roots,msb)
solve()
full_high=(6<<682-3)| base_val
p0=full_high<<342
print(p0.bit_length())

f=p0+3807104160372250427408151115969004938011834030077750047641248496672420141594744130350935075551156863041

from gmpy2 import*
q=n//f
phi=(f-1)*(q-1)
d=invert(e,phi)
m=pow(c,d,n)
long_to_bytes(int(m))
```
