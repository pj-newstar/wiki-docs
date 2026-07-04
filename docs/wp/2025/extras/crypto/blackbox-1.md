---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 黑盒 1

首先先猜黑盒结构是 $f(X)=SXS^{-1},X\in \mathbb{GL}(n,q)$，然后你就可以通过输入初等矩阵来求解了。

```python
n= 3
q=
F = GF(q)
i = 0
a = F(1)  # a=1

A = [0,0,0]
A[0]=[[2,0,0],[0,1,0],[0,0,1]]
A[1]=[[1,0,0],[1,1,0],[0,0,1]]
A[2]=[[1,0,0],[0,1,0],[1,0,1]]
A=[matrix(A[i]) for i in range(n)]
B = [0,0,0]
B[0]=
B[1]=
B[2]=
B=[matrix(Zmod(q),B[i]) for i in range(n)]

M0 = B[0] - matrix.identity(F, n)
k = None
for col in range(n):
    if not M0.column(col).is_zero():
        k = col
        break
if k is None:
    raise ValueError("无法找到非零列，请尝试不同的 i 或 a")

# 恢复 S 的列
S_columns = []
for j in range(n):
    M_j = B[j] - matrix.identity(F, n)
    u_j = M_j.column(k)  # 取第 k 列
    s_j = u_j / a  # 由于 a=1，可省略除法，但为了通用性保留
    S_columns.append(s_j)

# 将列组合成矩阵 S_recovered
S_recovered = matrix(F, n, n)
for j in range(n):
    S_recovered.set_column(j, S_columns[j])
```

如何判断黑盒结构是自同构？本题允许无限次输入 $X$ 并获取 $f(X)$，因此可以通过多次查询观察结构，再进行猜测与验证。

题目脚本：

```python

import secrets
import random
from secret import FLAG

# ---------- minimal finite-field matrix utilities ----------

def egcd(a,b):
    if b==0:
        return (a,1,0)
    g,x1,y1 = egcd(b, a % b)
    return (g, y1, x1 - (a//b) * y1)

def inv_mod(a, p):
    a %= p
    if a==0:
        raise ZeroDivisionError
    g,x,y = egcd(a,p)
    if g!=1:
        raise ZeroDivisionError
    return x % p

class Matrix:
    def __init__(self, rows, p):
        self.p = p
        self.rows = [[x % p for x in row] for row in rows]
        self.n = len(rows)

    @classmethod
    def zero(cls, n, p):
        return cls([[0]*n for _ in range(n)], p)

    @classmethod
    def identity(cls, n, p):
        M = cls.zero(n, p)
        for i in range(n):
            M.rows[i][i] = 1
        return M

    @classmethod
    def random_invertible(cls, n, p):
        while True:
            rows = [[secrets.randbelow(p) for _ in range(n)] for _ in range(n)]
            M = cls(rows, p)
            if M.det() % p != 0:
                return M

    def copy(self):
        return Matrix([row[:] for row in self.rows], self.p)

    def __mul__(self, other):
        if isinstance(other, Matrix):
            n=self.n; p=self.p
            R = [[0]*n for _ in range(n)]
            for i in range(n):
                for k in range(n):
                    aik = self.rows[i][k]
                    if aik==0: continue
                    rowk = other.rows[k]
                    ri = R[i]
                    for j in range(n):
                        ri[j] = (ri[j] + aik * rowk[j]) % p
            return Matrix(R, p)
        else:
            return Matrix([[ (a*other) % self.p for a in row] for row in self.rows], self.p)

    def scalar_mul(self, c):
        return Matrix([[ (c*e) % self.p for e in row] for row in self.rows], self.p)

    def det(self):
        n=self.n; p=self.p
        A = [row[:] for row in self.rows]
        det = 1
        for i in range(n):
            pivot = i
            while pivot < n and A[pivot][i] == 0:
                pivot += 1
            if pivot == n:
                return 0
            if pivot != i:
                A[i], A[pivot] = A[pivot], A[i]
                det = (-det) % p
            inv = inv_mod(A[i][i], p)
            det = (det * A[i][i]) % p
            for r in range(i+1, n):
                if A[r][i] == 0: continue
                factor = (A[r][i] * inv) % p
                for c in range(i, n):
                    A[r][c] = (A[r][c] - factor * A[i][c]) % p
        return det % p

    def inverse(self):
        n=self.n; p=self.p
        A = [row[:] for row in self.rows]
        I = [[1 if i==j else 0 for j in range(n)] for i in range(n)]
        for i in range(n):
            pivot = i
            while pivot < n and A[pivot][i] == 0:
                pivot += 1
            if pivot == n:
                raise ZeroDivisionError('singular')
            if pivot != i:
                A[i], A[pivot] = A[pivot], A[i]
                I[i], I[pivot] = I[pivot], I[i]
            inv = inv_mod(A[i][i], p)
            for c in range(n):
                A[i][c] = (A[i][c] * inv) % p
                I[i][c] = (I[i][c] * inv) % p
            for r in range(n):
                if r==i: continue
                factor = A[r][i]
                if factor==0: continue
                for c in range(n):
                    A[r][c] = (A[r][c] - factor * A[i][c]) % p
                    I[r][c] = (I[r][c] - factor * I[i][c]) % p
        return Matrix(I, p)

    def eq(self, other):
        return all((a-b) % self.p == 0 for ra,rb in zip(self.rows, other.rows) for a,b in zip(ra,rb))

    def to_string(self):
        return ';'.join(','.join(str(x) for x in row) for row in self.rows)

    @classmethod
    def from_string(cls, s, p):
        try:
            rows = [[int(x) % p for x in row.split(',')] for row in s.strip().split(';')]
            if len(rows) == 0: raise ValueError
            n = len(rows)
            if any(len(r)!=n for r in rows): raise ValueError
            return cls(rows, p)
        except Exception:
            raise ValueError('bad matrix format')

    def first_nonzero_entry(self):
        for i in range(self.n):
            for j in range(self.n):
                if self.rows[i][j] % self.p != 0:
                    return (i,j,self.rows[i][j] % self.p)
        return None

# ---------- challenge core ---------------------------------

class ConjugationChallenge:
    def __init__(self,FLAG, n=4, q=97, max_queries=200):
        self.n = n; self.q = q
        self.FLAG = FLAG
        self.max_queries = max_queries
        self.queries = 0
        self.S = Matrix.random_invertible(n, q)
        self.S_inv = self.S.inverse()
        self.X_secret = Matrix.random_invertible(n, q)
        self.Y_target = self.S * self.X_secret * self.S_inv
        self.stage1_passed = False

    def f(self, X: Matrix) -> Matrix:
        return self.S * X * self.S_inv

    def equal_up_to_scalar(self, A: Matrix, B: Matrix) -> bool:
        p = self.q
        entry = B.first_nonzero_entry()
        if entry is None:
            return A.first_nonzero_entry() is None
        i,j,b = entry
        a = A.rows[i][j]
        if a % p == 0:
            return False
        try:
            c = (a * inv_mod(b, p)) % p
        except ZeroDivisionError:
            return False
        for r in range(B.n):
            for ccol in range(B.n):
                if A.rows[r][ccol] % p != (c * B.rows[r][ccol]) % p:
                    return False
        return True

# ----------------- stdin/stdout loop ------------------------

def print_help():
    print('Whats:')
    print('  A black box (function). After inputting a matrix, it outputs a matrix.')
    print('  All matrices are in GL(n,q).')
    print('Commands:')
    print('  QUERY <matrix>      -- get f(matrix)')
    print('  STAGE1              -- get Y = f(X_secret); then guess X_secret')
    print('  GUESS <matrix>      -- submit candidate X for stage1')
    print('''  STAGE2              -- After you passed STAGE1 and figured out the structure of the black box,
                                      you need to submit a very important matrix S. f(X) is related to S.
                                      If you pass STAGE2, I will give you FLAG''')
    print('  SOLVE <matrix>      -- submit candidate S (accepted up to nonzero scalar)')
    print('  HELP                -- this message')
    print('  QUIT                -- exit')

def main():
    n_=random.randint(3,4)
    q_list=[97,18438074019027044383, 14812452124560918479, 17698650832589511139, 13688439759297834127, 12667571192190072239, 16736506655081653159, 13315982169682185221, 11120973245938580473, 17256530423800303043, 14121827379471213803]    #没想到吧，q 是有限的。
    q_=random.choice(q_list)
    maxq_list=[4,10,100]
    maxq=random.choice(maxq_list)
    chal = ConjugationChallenge(n=n_, q=q_, FLAG=FLAG, max_queries=maxq)
    print('Matrix blackbox (stdin/stdout). Type HELP.')
    try:
        while True:
            line = input('> ').strip()
            if not line:
                continue
            parts = line.split(' ',1)
            cmd = parts[0].upper(); arg = parts[1] if len(parts)>1 else ''
            if cmd == 'HELP':
                print_help()
                print('Parameters:')
                print(f'  n                   -- {chal.n}')
                print(f'  q                   -- {chal.q}')
                print(f'  max_queries         -- {chal.max_queries}')
                print('Matrix Input Demonstration:')
                print('  3x3 identity matrix -- 1,0,0;0,1,0;0,0,1')
            elif cmd == 'QUIT':
                print('bye')
                break
            elif cmd == 'QUERY':
                if chal.queries >= chal.max_queries:
                    print('ERROR: query limit reached')
                    continue
                try:
                    X = Matrix.from_string(arg, chal.q)
                except ValueError:
                    print('ERROR: bad matrix format')
                    continue
                Y = chal.f(X)
                chal.queries += 1
                print(Y.to_string())
            elif cmd == 'STAGE1':
                print('STAGE1: Here is Y = f(X_secret):')
                print(chal.Y_target.to_string())
                print('Now guess X_secret. You need send GUESS <matrix>')
            elif cmd == 'GUESS':
                try:
                    Xg = Matrix.from_string(arg, chal.q)
                except ValueError:
                    print('ERROR: bad matrix format')
                    continue
                if chal.f(Xg).eq(chal.Y_target):
                    print('Correct: stage1 passed. You can go to STAGE2.')
                    chal.stage1_passed = True
                else:
                    print('Incorrect preimage.')
            elif cmd == 'STAGE2':
                if not chal.stage1_passed:
                    print('You must pass stage1 first.')
                else:
                    print('Stage2: submit candidate using SOLVE <matrix>.')
            elif cmd == 'SOLVE':
                if not chal.stage1_passed:
                    print('You must pass stage1 first.')
                    continue
                try:
                    Sg = Matrix.from_string(arg, chal.q)
                except ValueError:
                    print('ERROR: bad matrix format')
                    continue
                if chal.equal_up_to_scalar(Sg, chal.S):
                    print('Correct! Here is your FLAG: ' + chal.FLAG)
                    break
                else:
                    print('Incorrect S.')
            else:
                print('Unknown command. Type HELP.')
    except EOFError:
        print('EOF - exit')

if __name__ == '__main__':
    main()

```
