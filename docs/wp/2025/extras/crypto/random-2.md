---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 随机数之旅 2

魔改 mt19937，但 temper 段简化了：

```python
  def extract_number(self):
    if self.mti == 0:
      self.twist()
    y = self.mt[self.mti]
    y = y ^ y >> 11                 # temper 段
    y = y ^ y << 7 & 0x0d000721     # temper 段
    self.mti = (self.mti + 1) % 114
    return _int32(y)

  def twist(self):
    for i in range(0, 114):
      y = _int32((self.mt[i] & 0x90000000) + (self.mt[(i + 1) % 114] & 0x8fffffff))
      self.mt[i] = (y >> 1) ^ self.mt[(i + 66) % 114]
      if y % 2 != 0:
        self.mt[i] = self.mt[i] ^ 0x0d000721
```

只需要逆向 temper 段，即可恢复完整的 `self.mt`，随后再输出 11 个随机数即可解密。

下面给出用于逆向 temper 段并恢复密钥的脚本：

```python
h=
c=

# ---- 工具函数
def xor(a,b):
  assert len(a)==len(b)
  return "".join(str((int(a[i])+int(b[i]))%2) for i in range(len(a)))

def both(a,b):
  assert len(a)==len(b)
  return "".join(str(int(a[i])*int(b[i])) for i in range(len(a)))

def _int32(x):
  return int(0xFFFFFFFF & x)

# 逆 temper 段第一行
def re1(y):
  y=bin(y)[2:].zfill(32)
  A = y[:11]
  B = xor(y[11:22],A)
  C = xor(y[-10:],B[:10])
  return int(A+B+C,2)

# 逆 temper 段第二行
def re2(y):
  t=bin(0x0d000721)[2:]
  y=bin(y)[2:].zfill(32)
  A = xor(y[-7:],both(t[-7:],"0"*7))
  B = xor(y[18:25],both(t[-14:-7],A))
  C = xor(y[11:18],both(t[-21:-14],B))
  D = xor(y[4:11],both(t[:7],C))
  E = y[:4]
  return int(E+D+C+B+A,2)

class MT19_937:
# 略

mt_rec=[re1(re2(_int32(h[i]))) for i in range(114)]

aa=MT19_937(0)
aa.mt=mt_rec
key=[aa.extract_number() for _ in range(11)]

x=1
for i in key:
  x*=i

from Crypto.Util.number import *
print(long_to_bytes(c^x))
```

补充说明：为什么恢复 `self.mt` 后可以直接输出 11 个随机数解密，而不需要先跳过 114 个数？

原因如下：

```python
  def extract_number(self):
    if self.mti == 0:
      self.twist()
      #......
```

`self.mti` 默认是0，当我们导入 `self.mt`（`aa.mt=mt_rec`）时，就已经帮我们过掉了。
