---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 置换 DLP

出题人测试的时候找规律做的。你也快来试试吧！

> [!NOTE] 找规律
> 我们举个简单的例子 ： $(12345)$, 我们观察它的幂都是什么样子：
> $(13524)$ >$(14253)$ >$(15432)$ >$\mathrm{id}$
> 另外一个例子：$(1234)$：
> $(13)(24)$ >$(1432)$ >$\mathrm{id}$
> 你可以猜一下$S_6$的元素会是什么情况。

题目脚本：

```python
import random, math, sys
from secret import FLAG

def lcm(a,b): return a* b // math.gcd(a,b)
def compose(a,b): return [a[i-1] for i in b]
def perm_pow(g,e):
    r=list(range(1,len(g)+1));
    while e:
        if e&1: r=compose(g,r)
        g=compose(g,g); e//=2
    return r
def cycles(p):
    seen=[0]*(len(p)+1); out=[]
    for i in range(1,len(p)+1):
        if not seen[i]:
            c=[]; j=i
            while not seen[j]: seen[j]=1; c.append(j); j=p[j-1]
            if len(c)>1: out.append("("+ " ".join(map(str,c)) +")")
    return " ".join(out) or "()"
def order(p):
    seen=[0]*(len(p)+1); res=1
    for i in range(1,len(p)+1):
        if not seen[i]:
            c=0; j=i
            while not seen[j]: seen[j]=1; j=p[j-1]; c+=1
            res=lcm(res,c)
    return res

def make_instance(force_unsolvable=False):
    n=random.randint(4,12)
    g=list(range(1,n+1))
    while g==list(range(1,n+1)) or order(g)<3: random.shuffle(g)
    ordg=order(g)
    if force_unsolvable:
        # pick random h not in <g>
        while True:
            h=list(range(1,n+1)); random.shuffle(h)
            if all(perm_pow(g,e)!=h for e in range(ordg)): break
    else:
        h=perm_pow(g,random.randrange(2,ordg))
    return n,g,h

def main():
    print("=== Permutation Discrete Log Challenge ===")
    print("Find x such that g^x = h (0 <= x < ord(g)), or print 'no' if no solution.")
    print("5 rounds total. One round has no solution.\n")
    unsolvable_round=random.randint(0,4)
    for r in range(5):
        n,g,h=make_instance(r==unsolvable_round)
        print(f"Round {r+1}/5\nS_{n}")
        print(cycles(g),"  ",cycles(h))
        ans=input("Your answer: ").strip()
        ordg=order(g)
        has_sol=any(perm_pow(g,e)==h for e in range(ordg))
        if ans.lower()=="no":
            if has_sol: print("Wrong. Solution exists."); return
        else:
            try: x=int(ans)
            except: print("Invalid input."); return
            if not has_sol or perm_pow(g,x%ordg)!=h:
                print("Wrong answer."); return
        print("Correct!\n")
    print("Congratulations! Your FLAG:",FLAG)

if __name__=="__main__":
    try: main()
    except (EOFError,KeyboardInterrupt): sys.exit(0)

```
