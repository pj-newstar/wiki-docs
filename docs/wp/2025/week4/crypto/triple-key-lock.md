---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 三重密钥锁

这道题是一个典型的隐藏子集和问题（Hidden Subset Sum Problem）。

$$k\times a + m\times b + n\times c \equiv f \pmod p$$

关键线索：a、b、c 都很小（只有 128 比特），而其他数都很大（512 比特）。

写成向量形式为：

$$(a, b, c, -t) \cdot (k, m, n, p) = f$$

为什么不能直接解方程？

这个方程看起来简单，但实际上是一个模方程，有无限多个解。在模 $p$ 的环境中，大数会把小数「淹没」，直接求解非常困难。

这里我们要了解一下「格」的概念。

什么是「格」？

可以将二维网格中的每个交叉点理解为「格点」。在数学上，格是由一组基向量生成的所有整数线性组合。

关键洞察：如果在高维空间中构造合适的格，那么目标小向量 $(a,b,c)$ 会成为该格中的一个「短向量」。

什么是「正交格」？

正交格就是与原始格中所有向量都垂直的格。

在本题中，需要找到向量 $(a, b, c, -t)$，使其与 $(k, m, n, p)$ 的点积为 $f$。

实际上，要找的是与目标向量 $(0,0,0,f)$ 很接近的格点。

这道题里，我们可以构造格基

```python
B = matrix(ZZ, [
    [1, 0, 0, k],
    [0, 1, 0, m],
    [0, 0, 1, n],
    [0, 0, 0, p]
])
```

这个矩阵的每一行都是一个基向量。由这些基向量生成的格包含了所有形如：

$(x, y, z, kx + my + nz + wp)$ 形式的向量，其中 $x,y,z,w$ 是整数。

那么如何找到与目标向量 $(0,0,0,f)$ 很接近的格点呢？

这里需要使用 Babai 最近向量算法。

Babai 算法的思想是：给定一个目标点，在格中寻找离它最近的点。

具体步骤如下：

1. 格基归约：先用 LLL 算法让格基更接近正交。
2. Gram-Schmidt 正交化：得到一组正交基。
3. 投影取整：将目标向量投影到正交基上，然后取整找到最近的格点。

```python
# 格基归约
B_red = B.LLL()
B_orth = B_red.gram_schmidt()[0]

# Babai 算法
w = target
for i in reversed(range(B_red.nrows())):
    mu = w.dot_product(B_orth[i]) / B_orth[i].dot_product(B_orth[i])
    w -= B_red[i] * round(mu)

solution = target - w
```

从找到的格点中提取前三个分量并拼接，即可得到 FLAG。

完整 EXP：

```python
from Crypto.Util.number import *
from sage.all import*

def solve_hssp(p, k, m, n, f):
    """使用正交格方法解决隐藏子集和问题"""
    print("=== 开始解隐藏子集和问题（正交格方法）===")

    # 构造格基
    # 我们有：k*a + m*b + n*c ≡ f (mod p)
    # 即：k*a + m*b + n*c + t*p = f，其中 t 是某个整数
    # 这可以写成：(a, b, c, -t) · (k, m, n, p) = f

    B = matrix(ZZ, [
        [1, 0, 0, k],
        [0, 1, 0, m],
        [0, 0, 1, n],
        [0, 0, 0, p]
    ])

    # 目标向量：我们想要找到接近(0,0,0,f)的格点
    target = vector(ZZ, [0, 0, 0, f])

    print("对格基进行 LLL 归约...")
    B_red = B.LLL()
    print("LLL 归约完成，进行 Gram-Schmidt 正交化...")
    B_orth = B_red.gram_schmidt()[0]

    # 使用 Babai 最近向量算法
    print("运行 Babai 最近向量算法...")
    w = target
    for i in reversed(range(B_red.nrows())):
        # 计算目标向量在当前正交基向量上的投影系数
        mu = w.dot_product(B_orth[i]) / B_orth[i].dot_product(B_orth[i])
        # 向最近的格点移动
        w -= B_red[i] * round(mu)

    # 计算最近格点与目标向量的差，这就是我们的解
    solution = target - w

    potential_a = abs(solution[0])
    potential_b = abs(solution[1])
    potential_c = abs(solution[2])

    # 验证解的正确性
    if (potential_a > 0 and potential_b > 0 and potential_c > 0 and
        potential_a.bit_length() <= 200 and
        potential_b.bit_length() <= 200 and
        potential_c.bit_length() <= 200 and
        (k * potential_a + m * potential_b + n * potential_c) % p == f):

        print(f"🎉 正交格方法找到解！")
        print(f"a = {potential_a}")
        print(f"b = {potential_b}")
        print(f"c = {potential_c}")
        return potential_a, potential_b, potential_c

    print("正交格方法未找到合适解")
    return None

# 解码 FLAG
def decode_flag(a, b, c):
    """从 a,b,c 解码 FLAG"""
    try:
        # 尝试不同的字节顺序
        for byte_order in ['big', 'little']:
            try:
                a_bytes = a.to_bytes((a.bit_length() + 7) // 8, byte_order)
                b_bytes = b.to_bytes((b.bit_length() + 7) // 8, byte_order)
                c_bytes = c.to_bytes((c.bit_length() + 7) // 8, byte_order)

                flag_bytes = a_bytes + b_bytes + c_bytes
                FLAG = flag_bytes.decode('ascii', errors='ignore')

                if 'FLAG{' in FLAG:
                    return FLAG
            except:
                continue
        return None
    except:
        return None

# 题目参数
p = 10424356578148041779853991789187969944186570125402901113699573185144158488847151089093649435805832723680640302469301322004769382556869280204369016044400623
k = 2016425917343526209264752974016973527106088400191647819396444997081866888816818440804306653900752825844532111319244334210470353279795203950886189568717273
m = 9640575609666038466312358795458735166723157003124018050805657432015561577987823522956739610343817276374800232163184447140344754253531140765054930193240661
n = 8539207304708818916453730202381072788689351891251165656488809155919585187699733568697903825636944248694317545906020707873051567183468920809837554174735591
f = 3760813688323379339493776734416231127517302841171887658445242754803946122769018586447782634756726656702581791734772105099204609201876825961922712387326893

# 主求解流程
print("开始求解三重密钥锁问题...")
print(f"已知参数: p={p}")
print(f"系数 k={k}")
print(f"系数 m={m}")
print(f"系数 n={n}")
print(f"验证值 f={f}")

a_sol, b_sol, c_sol = solve_hssp(p, k, m, n, f)

if a_sol is not None:
    print("\n=== 验证解 ===")
    computed_f = (k * a_sol + m * b_sol + n * c_sol) % p
    print(f"计算: k*a + m*b + n*c mod p = {computed_f}")
    print(f"目标: f = {f}")
    print(f"验证通过: {computed_f == f}")

    # 解码 FLAG
    flag_found = decode_flag(a_sol, b_sol, c_sol)
    if flag_found:
        print(f"\n🎉 成功找到 FLAG: {flag_found}")
    else:
        print("找到 a,b,c 但解码 FLAG 失败，尝试手动解码...")
        print(f"a = {a_sol}")
        print(f"b = {b_sol}")
        print(f"c = {c_sol}")
else:
    print("未能找到解")
```
