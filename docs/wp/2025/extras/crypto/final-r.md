---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# final_R

神秘一行 python

解释：FLAG 转为$\mathbb{GF}(2^7)$上的（$59$维）向量（通过对每个字符使用 ord()，值 $x$ 代表 $a^x$ ,$a$ 是生成元）. 然后该向量对自己做循环卷积。结果向量（的值$a^x\in \mathbb{GF}(2^7)$转为$x$，）输出为字节形式。

你当然可以直接解卷积。

商环 $\mathbb{F}[x]/(x^n-1)$ 和向量空间 $\mathrm{F}^n$ 有一一对应关系：
$r(x)=\sum_i a_ix^{i}\in \mathbb{F}[x]/(x^n-1) \rightarrow (\cdots,a_i,\cdots)\in \mathbb{F}^n$
多项式加法对应向量加法
多项式乘法对应向量的循环卷积。

所以本题其实就是求一个多项式的平方根。
怎么求呢？

非常简单，直接列出来观察就行。因为$\mathbb{GF}(2^7)$ 特征是 $2$ ，所以没有交叉项。直接对应系数相等求解就可以了。

```python
# 解题脚本。
n = 59
char = 2
m = 7

F = GF(2**m, 'a')
a = F.gen() # 获取本原元 a

R.<x> = F[]
mod_poly = x**n - 1

def find_sqrt_in_quotient_ring(f_poly):
    f_coeffs = f_poly.coefficients(sparse=False)
    if len(f_coeffs) < n:
        f_coeffs.extend([F(0)] * (n - len(f_coeffs)))
    inv_2 = pow(2, -1, n)

    g_coeffs = []
    for k in range(n):
        source_index = (2 * k) % n
        a_j = f_coeffs[source_index]
        if a_j == 0:
            b_k = F(0)
        else:
            b_k = a_j**64
        g_coeffs.append(b_k)
    g_poly = R(g_coeffs)
    return g_poly

cc=b'MfYGCnO`w%\x07zSzejG#kkb\x01\x01%eS?]GO`?]\x03m?`ab`kbnsS]``][?S`C\x1dB?{m'
cc=[a**i for i in cc]
f =R(cc)

g = find_sqrt_in_quotient_ring(f)
mv = [c.log(a) for c in g.coefficients(sparse=False)]
mm="".join(chr(mv[i]) for i in range(len(mv)))
print(mm.encode())
```
