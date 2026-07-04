---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# Poly

<Container type='info'>

本题考查 Groebner 基。
</Container>

参考 EXP：

```python

def l2b(n: int, length: int = None) -> bytes:
# 略

def b2l(b: bytes) -> int:
# 略

#--------  题目脚本   ------
import uuid
p = random_prime(2**256)
f=f"FLAG{{{uuid.uuid4()}}}"
x1=f[:len(f)//2]
x2=f[len(f)//2:]
m1 = b2l(x1.encode())
m2 = b2l(x2.encode())
c1 = (m1^19+m1^18+4*m1^17)%p
c2 = (5*m2^19+m2^18+4*m2^17)%p
s = (m1^7*m2^2+m2)%p
print((p,c1,c2,s))

#--------  解题脚本  ---------
R = PolynomialRing(Zmod(p), 'x, y')
x, y = R.gens()
I = Ideal([x^19+x^18+4*x^17-c1, 5*y^19+y^18+4*y^17-c2, x^7*y^2+y-s])
for g in I.groebner_basis():
    print(g)
    a=Zmod(p)(-g.univariate_polynomial()(0))
    print(l2b(int(a)))
```

简单来说，groebner基可以用来求多项式的公因式。

Gröbner基 方法是从任意一个多项式理想的一组给定生成元，计算另一组性质良好的生成元，并称为该理想的Gröbner基。它能用来判定任意多项式是否属于该理想。做密码题时，则使用该基简化运算来求解我们想要的。
