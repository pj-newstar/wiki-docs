---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 随机数之旅 1.3

```python
import uuid
from Crypto.Util.number import *
import random

FLAG="FLAG{"+str(uuid.uuid4())+"}"
m=bytes_to_long(FLAG.encode())

p=getPrime(m.bit_length()+3)
a=getPrime(p.bit_length())

print("p=",p)

hint=[random.randint(1,p-1),]
for i in range(10):
  hint.append((a*hint[-1]+m)%p)

print(hint)
```

类似随机数之旅1，但这里只给了$p$，需要我们自己求$a,m$
思路见随机数之旅1（w1），GCL(w3)

```python
from Crypto.Util.number import *

p=
h=

a=(h[4]-h[3])*inverse(h[3]-h[2],p) % p
b=(h[3]-a*h[2])%p

print(long_to_bytes(b))
```
