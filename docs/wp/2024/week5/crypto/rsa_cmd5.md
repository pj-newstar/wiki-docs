---
titleTemplate: ":title | WriteUp - NewStar CTF 2024"
---

<script setup>
import Container from '@/components/docs/Container.vue'
import { ElCollapse, ElCollapseItem } from 'element-plus'
import 'element-plus/es/components/collapse/style/css'
import 'element-plus/es/components/collapse-item/style/css'

const vProps = {
  openCollapse: []
}
</script>

# RSA? cmd5!

## 题目信息

<Container type='quote' title='出题思路'>

RSA 的签名，往往是给接收方 B 确定发送方 A 的身份用的，但如果发送方用不安全的哈希算法来计算信息的信息摘要，那么有可能被中途的窃密者 C 利用而得到明文。
</Container>

题目描述：

> 应该都了解 RSA 解密原理了吧？
>
> **Alice:** 你把服务器密码用 RSA 加密给我发过来吧，记得带上签名.
>
> **Bob:** 好麻烦，还要签名，算了，直接 MD5 签名一下就好了吧！

<ElCollapse class='vp-collapse' v-model='vProps.openCollapse'>
  <ElCollapseItem  class='no-border' name='attachment-content'>
  <template #title>
    题目附件内容<span data-desc v-text='vProps.openCollapse.includes("attachment-content") ? "（点此收起）" : "（点此展开）"'></span>：
  </template>

::: code-group

```python [task.py]
from Crypto.Util.number import *
from gmpy2 import *
import hashlib

# 什么？你说你用 MD5 给 RSA 签名了？！

m = '*******'
assert len(m) == 7
flag = 'flag{th1s_1s_my_k3y:' + m + '0x' + hashlib.sha256(m.encode()).hexdigest() + '}'

p = getStrongPrime(512)
q = getStrongPrime(512)
n = p * q
e = 65537
phi = (p - 1) * (q - 1)
d = inverse(e, phi)


def get_MD5(m0):
    import hashlib
    md5_object = hashlib.md5(m0.encode())
    md5_result = md5_object.hexdigest()
    return md5_result


def get_s(m0, d0, n0):
    hm0 = get_MD5(m0)
    hm1 = bytes_to_long(hm0.encode())
    s0 = pow(hm1, d0, n0)
    return s0


def rsa_encode(m0, e0, n0):
    m1 = bytes_to_long(m0.encode())
    c0 = pow(m1, e0, n0)
    return c0


def get_flag(m0):  # 请用这个函数来转m得到flag
    import hashlib
    flag = 'flag{th1s_1s_my_k3y:' + m0 + '0x' + hashlib.sha256(m0.encode()).hexdigest() + '}'
    print(flag)


s = get_s(m, d, n)
c = rsa_encode(flag, e, n)

print("密文c =", c)
print("签名s =", s)
print("公钥[n,e] =", [n, e])

'''
密文c = 119084320846787611587774426118526847905825678869032529318497425064970463356147909835330423466179802531093233559613714033492951177656433798856482195873924140269461792479008703758436687940228268475598134411304167494814557384094637387369282900460926092035234233538644197114822992825439656673482850515654334379332
签名s = 5461514893126669960233658468203682813465911805334274462134892270260355037191167357098405392972668890146716863374229152116784218921275571185229135409696720018765930919309887205786492284716906060670649040459662723215737124829497658722113929054827469554157634284671989682162929417551313954916635460603628116503
公钥[n,e] = [139458221347981983099030378716991183653410063401398496859351212711302933950230621243347114295539950275542983665063430931475751013491128583801570410029527087462464558398730501041018349125941967135719526654701663270142483830687281477000567117071676521061576952568958398421029292366101543468414270793284704549051, 65537]
'''
```

:::

  </ElCollapseItem>
</ElCollapse>

## 解析

这里我们得到了 $c$、$s$、$n$、$e$，和平时的 RSA 不同，这里多了 $s$，让我们看看这个 $s$ 是什么

```python
def get_MD5(m0):
    import hashlib
    md5_object = hashlib.md5(m0.encode())  # 创建 MD5 对象并对文本进行编码
    md5_result = md5_object.hexdigest()  # 获取加密结果
    return md5_result


def get_s(m0, d0, n0):
    hm0 = get_MD5(m0)
    hm1 = bytes_to_long(hm0.encode())
    s0 = pow(hm1, d0, n0)
    return s0
```

计算了 `m0` 的 MD5 为变量 `hm0`，然后将变量 `hm0` 转成整数 `hm1` 进行计算

$$
s = hm_1^{d} \mod n
$$

那么我们有 $e$，理解 RSA 解密原理

$$
\begin{gather}
e \cdot d \equiv 1 \mod \varphi \\
s^e \equiv (hm_1^{d})^e \equiv hm_1^{d \cdot e} \equiv hm_1^{1+k \cdot \varphi} \equiv hm_1 \mod n = hm_1
\end{gather}
$$

那么再结合题目名字和描述，去 [cmd5.com](https://cmd5.com) 找 `hm1` 的结果:

![CMD5 网站](/assets/images/wp/2024/week5/rsa_cmd5_1.png)

$m$ 为 `adm0n12`

计算 flag

```python
flag = 'flag{th1s_1s_my_k3y:' + m + '0x' + hashlib.sha256(m.encode()).hexdigest() + '}'
```

## EXP

```python
from Crypto.Util.number import *
from gmpy2 import *

s = 7270513885194445005322518350289419893608325839878215682947885852347014936106128407554345668066935779849573932055239642406851308417046145495939362638652861562381316163080735160853285303356461796079298817982074998651099375222398758502559657988024308504098238446594559605603104540325738607539729848183025647146
e = 65537
n = 118167767283404647838357773955032661171703847685597271116789633496884884504237966404005641401909577369476550625894333528860763752286157264860218284704704444830864099870199623580368198306940575628872723737071517733553706154898255520538220530675603850372384339470410704813339357637359108745206967929184573003377


def get_flag(m0):  # 请用这个函数来转m得到flag
    import hashlib
    flag = 'flag{th1s_1s_my_k3y:' + m0 + '0x' + hashlib.sha256(m0.encode()).hexdigest() + '}'
    print(flag)


hm_int = pow(s, e, n)
hm_str = long_to_bytes(hm_int).decode()
print(hm_str)  # 86133884de98baada58a8c4de66e15b8

m = 'adm0n12'  # 通过 cmd5.com 查询到的结果
get_flag(m)
# flag{th1s_1s_my_k3y:adm0n120xbfab06114aa460b85135659e359fe443f9d91950ca95cbb2cbd6f88453e2b08b}
```

神秘数字：

```plaintext
33020419294068288558474507763458223726290143000598988310562749956226420170565399347815795
```
