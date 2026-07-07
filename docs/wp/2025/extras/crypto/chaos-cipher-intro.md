---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 混沌密码学入门

题目已经给出混沌序列和加密过程，因此按加密流程逆向恢复即可。

EXP：

```python
from math import sin,pi
from PIL import Image

def Feigenbaum_Equation(a,b,r1,r2,x1,y1,n,t):
    x=[x1]
    y=[y1]
    for _ in range(n):
        x.append(3*a*sin(pi*x[-1])+r1*y[-1])
        y.append(3*b*sin(pi*y[-1])+r2*y[-1])
    return x[-t:],y[-t:]

img = Image.open(r"\chaos_chaos.png")
pixels = img.load()
w, h = img.size
n = w * h

xl,_=Feigenbaum_Equation(0.9,1.01,0.1,0.2,0.22,0.43,2*n,n)

pixel_list = []
for x in range(w):
    for y in range(h):
        pixel_list.append(pixels[x, y])

# 下面是通过两次索引数组排序恢复像素的正确位置

N=[[0]*n for _ in range(2)]
N[0]=[i+1 for i in range(n)]
N[1]=xl

N1i = sorted(enumerate(N[1]), key=lambda x: x[1])
Ni = [index for index, value in N1i]
N_ = []
for row in N:
    new_row = [row[i] for i in Ni]
    N_.append(new_row)

xn=N_[0]

L=[[0]*n for _ in range(2)]
L[0]=xn
L[1]=pixel_list

L1i = sorted(enumerate(L[0]), key=lambda x: x[1])
Li = [index for index, value in L1i]
L_ = []
for row in L:
    new_row = [row[i] for i in Li]
    L_.append(new_row)

index = 0
for x in range(w):
    for y in range(h):
        pixels[x, y] = L_[1][index]
        index += 1

img.show()
```

解密结果：

> 此处原文引用了示意图，但图片文件未包含在压缩包中。

由于浮点精度、舍入和误差累积问题，Windows 与 Linux 生成的混沌序列在约 30 个数后会出现明显差异，因此不同平台的解密结果可能不同。
