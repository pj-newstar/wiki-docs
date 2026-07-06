---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 独一无二

<Container type='info'>

本题考查 ECDSA。
</Container>

题目关键在于复用了随机数 $k$ ，这导致你可以计算出私钥 $D$

```python
from Crypto.Util.number import bytes_to_long as b2l
from Crypto.Util.Padding import pad
from Crypto.Cipher import AES
from sympy import prevprime
import uuid
import random
import os

d=os.urandom(16)
D=b2l(d)     #签名的私钥 D
FLAG = f"FLAG{{{uuid.uuid4()}}}"
cipher=AES.new(d,AES.MODE_ECB)
ct=cipher.encrypt(pad(FLAG.encode(),16))
print("ct=",ct)

mes1=b"If you used the same random number when signing,"
mes2=b" then you need to be careful."
e1, e2 = b2l(mes1), b2l(mes2)

p=random_prime(2**128)
A,B=random.randint(1,p-1),random.randint(1,p-1)
E = EllipticCurve(Zmod(p),[A, B])
G=E.gens()[0]
n = prevprime(E.order())
print("n=",n)

k=random.randint(1,n-1)
Q=k*G
r=int(Q[0])%n
k_inv = pow(k, -1, n)
assert r!=0

s1 = (k_inv * (e1 + r * D)) % n
s2 = (k_inv * (e2 + r * D)) % n    #这里复用了随机数 k
print("(r1,s1)=",(r,s1))
print("(r2,s2)=",(r,s2))
```

```python
#EXP

ct=
n=
(r1,s1)=
(r2,s2)=

from Crypto.Cipher import AES
from Crypto.Util.number import long_to_bytes,bytes_to_long as b2l

mes1=b"If you used the same random number when signing,"
mes2=b" then you need to be careful."
e1=b2l(mes1)
e2=b2l(mes2)

k=(e1-e2)*pow(s1-s2,-1,n)%n   #通过关系式求解出 k
d=(s1*k-e1)*pow(r1,-1,n)%n    #通过 k 求解出私钥 D

d=long_to_bytes(int(d))
cipher=AES.new(d,AES.MODE_ECB)
pt=cipher.decrypt(ct)
print(pt)

```
