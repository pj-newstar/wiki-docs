---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 随机数之旅 3.6

```python
# Sage 9.3
import random
import uuid

FLAG="FLAG{"+str(uuid.uuid4())+"}"
n=len(FLAG)
m=n-6
p=random_prime(2**64)

A=[random.randint(p//2,p-1) for _ in range(m*n)]
A=matrix(Zmod(p),m,n,A)
x=[ord(i) for i in FLAG]
x=vector(x)
b=A*x

with open("output.txt","w") as f:
  f.write(str(p)+"\n")
  f.write(str(list(A))+"\n")
  f.write(str(list(b)))
```

这是「随机数之旅 3」的进阶版本。这里 $m=n-6$，方程数量不足，但 FLAG 的六个已知字符 `f,l,a,g,{,}` 可以用于补充约束并求解。

```python
# Sage
with open("output.txt","r") as f:
    p=eval(f.readline())
    A=eval(f.readline())
    b=eval(f.readline())

A=matrix(Zmod(p),A)
b=vector(b)

x0=A.solve_right(b)  # 特解

ArK=A.right_kernel().basis()   #右核的基
c=[ArK[i] for i in range(6)]

known = {0:ord('f'),1:ord('l'),2:ord('a'),3:ord('g'),4:ord('{'),-1:ord('}')}
rows = []
rhs = []
for idx, val in known.items():
    row = [c_i[idx] for c_i in ArK]   # 取 6 个基向量在该坐标的分量
    rows.append(row)
    rhs.append(val - x0[idx])
C = Matrix(Zmod(p), rows)
Y = vector(Zmod(p), rhs)
T = C.solve_right(Y)   #计算系数

mes=x0+sum(T[i]*c[i] for i in range(6))
print(mes)
mes=list(mes)
f="".join(chr(int(mes[i])) for i in range(len(mes)))
print(f)
```

预期外解法：格规约。

```python
n=42
k=16   #取 16 条方程就足够计算了。
Ge=matrix(ZZ,n+2*k,n+2*k)
for i in range(n):
    Ge[i,i]=1#+1
    #Ge[n,i]=-1
    for j in range(k):
        Ge[n+j,n+j]=1
        Ge[i,n+k+j]=A[j][i]
        Ge[n+j,n+k+j]=-p
        Ge[n+k+j,n+k+j]=b[j]

Ge=Ge.LLL()
print(Ge)
```
