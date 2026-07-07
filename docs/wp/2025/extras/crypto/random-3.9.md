---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 随机数之旅 3.9

$A_{500\times 30}\pmb x_{30\times 1}+\pmb e_{500\times 1}=\pmb b_{500\times 1}$
已知 $A,\pmb b$，且 $\pmb e$ 的分量只有两种取值，要求恢复 $\pmb x$。

```python
n=30
m=500
p=random_prime(2**64)

ec=[random.randint(1,p-1) for _ in range(2)]
e=[random.choice(ec) for _ in range(m)]
e=vector(e)

A=[random.randint(1,p-1) for _ in range(m*n)]
A=matrix(Zmod(p),m,n,A)
x=vector([random.randint(1,2**32) for _ in range(n)])

b=A*x+e
```

显然有
$$\sum_{j}^{30}a_{ij}x_j +e_i=b_i$$
于是
$$(\sum_{j}^{30}a_{ij}x_j +ec[0]-b_i)(\sum_{j}^{30}a_{ij}x_j +ec[1]-b_i)=0$$
这样的二次方程共有 500 个，而未知项数量为 $30\times 31/2+30=495$，因此可以求解。
将 $x_0x_0,x_0x_1,\cdots,x_ix_j,\cdots,x_{29}x_{29},x_0,\cdots,x_{29}$ 视为独立未知量后，该问题可转化为线性方程组。

```python
n=30
m=500
t=30*31/2+30

C=matrix(Zmod(p),m,t)
Y=[0 for _ in range(m)]
for i in range(m):
    for j in range(n):
        C[i,j]=A[i][j]*(ec[0]+ec[1])-2*A[i][j]*b[i]

for k in range(m):
    for i in range(n):
        for j in range(i+1):
            C[k,n+i*(i+1)//2+j]=A[k][i]*A[k][j]

for i in range(m):
    Y[i]=-1*(ec[0]-b[i])*(ec[1]-b[i])

Y=vector(Y)
print(C.rank())#495
xx=C.solve_right(Y)
print(xx[:n])

key=prod(xx[:n])
```

也可以考虑使用 NumPy 优化矩阵运算，但当前规模下直接求解已经足够。

意料之中的非预期：
使用线性映射 $ec[i]\rightarrow a\cdot ec[i]+b=ec^{\prime}[i]$，让 $ec^{\prime}[i]$ 足够小，然后用格方法求解。

意料之外的非预期：
中间相遇攻击
