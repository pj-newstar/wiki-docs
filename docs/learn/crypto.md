---
titleTemplate: ":title | 快速入门 - NewStar CTF"
---

<script setup>
import Container from '@/components/docs/Container.vue'
import Link from '@/components/docs/Link.vue'
import { ElTag, ElCollapse, ElCollapseItem, ElTooltip } from 'element-plus'
import 'element-plus/es/components/tag/style/css'
import 'element-plus/es/components/collapse/style/css'
import 'element-plus/es/components/collapse-item/style/css'
import 'element-plus/es/components/tooltip/style/css'

const vProps = {
  openCollapse: []
}
</script>

# 密码学

简单来说，密码学是一种用来研究如何加密来保护信息，如何解密获取信息的学问。

## 为什么学密码

可以了解、开创并实现现代密码学的 RSA 算法、未来将抵御量子计算机的格密码等的「后量子密码技术」，以及学到在特殊情况下我们用现代计算机来破解特定的 RSA 现代密码和后量子密码技术的攻击手段。

密码有清晰的路线学习：

![现代密码学](/assets/images/learn/crypto-instruction.png)

并且有干脆的做题反馈，就跟数学题一样，会就是会，不会就是不会。

或者说没有理由，当你入门密码学的时候，就会发现做密码题也会上瘾！

## 准备工作

你需要准备以下工具：

- PC<span data-desc>（一般不限操作系统）</span>
- Python <ElTag type="primary" size="small">编程语言和环境</ElTag>
- yafu <ElTag type="primary" size="small">软件程序</ElTag>
- 一款笔记软件或网站
- 草稿纸和笔<span data-desc>（随电脑携带的，经常需要演算关系式）</span>
- <Link icon="external" theme="underline hover" href="https://factordb.com/">大数分解 factordb</Link> <ElTag type="primary" size="small">网站</ElTag>

其它一些帮助性较大的网站：

- <Link icon="external" theme="underline hover" href="https://www.nssctf.cn/explore/workshop/">NSSCTF 探索商城</Link>
- <Link icon="external" theme="underline hover" href="https://ctf-wiki.org/">CTF Wiki</Link>
- <Link icon="external" theme="underline hover" href="https://oi-wiki.org/">OI Wiki</Link><span data-desc>（包含一些更深的数学知识）</span>

除此之外，在学习的途中还会遇到很多有意思的工具或网站。

## 学习路线

### Python 编程基础

下载 Python，选择一款集成开发环境（IDE，如 [PyCharm](https://www.jetbrains.com/pycharm/) 或 [VSCode](https://code.visualstudio.com/)），并配置好 Python 环境。

由于国内使用 Python 的包管理工具下载缓慢，可参照 [清华大学软件源 PyPI](https://mirror.tuna.tsinghua.edu.cn/help/pypi/) 配置镜像源。

使用 Python 的包管理工具安装必要的密码学库：

```bash
pip install pycryptodome
pip install gmpy2
```

在 Python 的学习中，你必须了解：

- 变量和变量类型，变量类型转换
- 运算符号
- 基本语法
- 循环语句
- 布尔类型
- 列表和元组
- 模块导入
- 函数的使用
- Python 中各种进制的格式头

你可以通过解释如下代码片段中高亮行的含义来检验自己的学习情况<span data-desc>（下面的代码整体无实际意义）</span>：

```python:line-numbers
from Crypto.Util.number import * # [!code highlight]
import gmpy2

flag = b"flag{test_flag}"
flag = bytes_to_long(flag) # [!code highlight]

e = b'65537'
e = int(a) # [!code highlight]

plist = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29] # [!code highlight]
plist = tuple(plist) # [!code highlight]

p = getPrime(256)
q = 0xda1a92b4bf451ce51f14a1d178e224a237f3e7e4b18a60084ae7da91889830b1 # [!code highlight]
n = p*q

c = pow(flag, e, n) # [!code highlight]

x = c % p // 2 # [!code highlight]
```

### 密码学基础

你需要了解下面这些知识点：

- 理解模运算
- 逆元计算的运用<span data-desc>（暂时不要求理解）</span>
- RSA 公钥加密算法<span data-desc>（可参考文章：[RSA 入门](https://dexterjie.github.io/2024/07/15/非对称加密/RSA/)）</span>
- 理解海纳百川的 $k$ 和无所不能的公因数

## 开始学习

### （一）掌握一定语言基础

首先建议学 Python，密码题经常出现很大的数字和较难的运算，靠手算很难得到答案，所以需要一门计算机语言来帮我们计算。而 Python 是较符合人类直觉并且易用的语言，可以快速编写实现一个功能，对于新手而言最为友好，故推荐从 Python 入门编程语言。

对于学习 Python，了解以下知识点即可（摘自 [Python 基础教程 | 菜鸟教程](https://www.runoob.com/python)）。

<ElCollapse class='vp-collapse' v-model='vProps.openCollapse'>
<ElCollapseItem name='acknowledge-list'>
<template #title>
  <strong>知识点列表</strong><span data-desc v-text='vProps.openCollapse.includes("acknowledge-list") ? "（点此收起）" : "（点此展开）"'></span>
</template>

- Python 基础教程
- Python 简介
- Python 环境搭建
- Python 中文编码
- Python 基础语法
- Python 变量类型
- Python 运算符
- Python 条件语句
- Python 循环语句
- Python While 循环语句
- Python for 循环语句
- Python 循环嵌套
- Python break 语句
- Python continue 语句
- Python pass 语句
- Python Number（数组）
- Python 字符串
- Python List（列表）
- Python 日期和时间
- Python 函数
- Python 模块

</ElCollapseItem>
</ElCollapse>

::: tip 撰稿人注
讲讲我个人学习 Python 的经历：我一开始就买了本 Python 的入门书籍开始「啃」，啃到后面 for 循环就开始做题了。一开始也不会做题，就看别人的 WriteUp（看看别人的思路，抄作业，但这是个好的行为，大伙开始都是看过来的），WriteUp 里一般都会有 Python 代码，就跟着打在自己的电脑上（建议不要直接复制粘贴），遇到不会的就上网搜索，一边学密码学，一边练编程技术。
:::

::: tip 编者注
编者具有其它程序语言的基础，对 Python 也只随便看了一些基本的语法（运算、循环）之后便上手密码学了。刚开始做题很艰难，几乎写每一行功能性代码都要网上去搜索（诸如用什么库、函数），经历过大约半个月的持续边做题边编程、看 WriteUp、赛后与前辈交流，Python 能力也见长了。遇到不会的立即学习并应用，慢慢地也就无师自通了。
:::

### （二）了解 RSA 算法（公钥加密算法）

<Container type='tip'>
「毫不夸张地说，只要有计算机网络的地方，就有 RSA 算法。」

<div style="display: inline-block; width: 100%; text-align: right;">——阮一峰</div>
</Container>

#### 1. 元素

RSA 算法里面一共有 8 个元素。

- $p$，一个素数。
- $q$，另一个素数
- $n$，根据 $p$ 和 $q$ 形成的数，一般为 $n = p \times q$
- $e$，一个被称为「公钥因子」的数
- $d$，一个被称为「私钥因子」的数。下面的公式会具体展现e和d的转化
- $\varphi$，一个与 $n$ 有关，由 $p$ 和 $q$ 计算而得到的数。具体就是 $n$ 的欧拉函数 <ElTooltip content='<mjx-container class="MathJax" jax="SVG" style="direction: ltr; position: relative;"><svg xmlns="http://www.w3.org/2000/svg" width="21.107ex" height="2.262ex" role="img" focusable="false" viewBox="0 -750 9329.4 1000" aria-hidden="true" style="overflow: visible; min-height: 1px; min-width: 1px; vertical-align: -0.566ex;"><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="scale(1,-1)"><g data-mml-node="math"><g data-mml-node="mi"><path data-c="1D711" d="M92 210Q92 176 106 149T142 108T185 85T220 72L235 70L237 71L250 112Q268 170 283 211T322 299T370 375T429 423T502 442Q547 442 582 410T618 302Q618 224 575 152T457 35T299 -10Q273 -10 273 -12L266 -48Q260 -83 252 -125T241 -179Q236 -203 215 -212Q204 -218 190 -218Q159 -215 159 -185Q159 -175 214 -2L209 0Q204 2 195 5T173 14T147 28T120 46T94 71T71 103T56 142T50 190Q50 238 76 311T149 431H162Q183 431 183 423Q183 417 175 409Q134 361 114 300T92 210ZM574 278Q574 320 550 344T486 369Q437 369 394 329T323 218Q309 184 295 109L286 64Q304 62 306 62Q423 62 498 131T574 278Z" style="stroke-width:3;"></path></g><g data-mml-node="mo" transform="translate(654,0)"><path data-c="28" d="M94 250Q94 319 104 381T127 488T164 576T202 643T244 695T277 729T302 750H315H319Q333 750 333 741Q333 738 316 720T275 667T226 581T184 443T167 250T184 58T225 -81T274 -167T316 -220T333 -241Q333 -250 318 -250H315H302L274 -226Q180 -141 137 -14T94 250Z" style="stroke-width:3;"></path></g><g data-mml-node="mi" transform="translate(1043,0)"><path data-c="1D45B" d="M21 287Q22 293 24 303T36 341T56 388T89 425T135 442Q171 442 195 424T225 390T231 369Q231 367 232 367L243 378Q304 442 382 442Q436 442 469 415T503 336T465 179T427 52Q427 26 444 26Q450 26 453 27Q482 32 505 65T540 145Q542 153 560 153Q580 153 580 145Q580 144 576 130Q568 101 554 73T508 17T439 -10Q392 -10 371 17T350 73Q350 92 386 193T423 345Q423 404 379 404H374Q288 404 229 303L222 291L189 157Q156 26 151 16Q138 -11 108 -11Q95 -11 87 -5T76 7T74 17Q74 30 112 180T152 343Q153 348 153 366Q153 405 129 405Q91 405 66 305Q60 285 60 284Q58 278 41 278H27Q21 284 21 287Z" style="stroke-width:3;"></path></g><g data-mml-node="mo" transform="translate(1643,0)"><path data-c="29" d="M60 749L64 750Q69 750 74 750H86L114 726Q208 641 251 514T294 250Q294 182 284 119T261 12T224 -76T186 -143T145 -194T113 -227T90 -246Q87 -249 86 -250H74Q66 -250 63 -250T58 -247T55 -238Q56 -237 66 -225Q221 -64 221 250T66 725Q56 737 55 738Q55 746 60 749Z" style="stroke-width:3;"></path></g><g data-mml-node="mo" transform="translate(2309.8,0)"><path data-c="3D" d="M56 347Q56 360 70 367H707Q722 359 722 347Q722 336 708 328L390 327H72Q56 332 56 347ZM56 153Q56 168 72 173H708Q722 163 722 153Q722 140 707 133H70Q56 140 56 153Z" style="stroke-width:3;"></path></g><g data-mml-node="mo" transform="translate(3365.6,0)"><path data-c="28" d="M94 250Q94 319 104 381T127 488T164 576T202 643T244 695T277 729T302 750H315H319Q333 750 333 741Q333 738 316 720T275 667T226 581T184 443T167 250T184 58T225 -81T274 -167T316 -220T333 -241Q333 -250 318 -250H315H302L274 -226Q180 -141 137 -14T94 250Z" style="stroke-width:3;"></path></g><g data-mml-node="mi" transform="translate(3754.6,0)"><path data-c="1D45D" d="M23 287Q24 290 25 295T30 317T40 348T55 381T75 411T101 433T134 442Q209 442 230 378L240 387Q302 442 358 442Q423 442 460 395T497 281Q497 173 421 82T249 -10Q227 -10 210 -4Q199 1 187 11T168 28L161 36Q160 35 139 -51T118 -138Q118 -144 126 -145T163 -148H188Q194 -155 194 -157T191 -175Q188 -187 185 -190T172 -194Q170 -194 161 -194T127 -193T65 -192Q-5 -192 -24 -194H-32Q-39 -187 -39 -183Q-37 -156 -26 -148H-6Q28 -147 33 -136Q36 -130 94 103T155 350Q156 355 156 364Q156 405 131 405Q109 405 94 377T71 316T59 280Q57 278 43 278H29Q23 284 23 287ZM178 102Q200 26 252 26Q282 26 310 49T356 107Q374 141 392 215T411 325V331Q411 405 350 405Q339 405 328 402T306 393T286 380T269 365T254 350T243 336T235 326L232 322Q232 321 229 308T218 264T204 212Q178 106 178 102Z" style="stroke-width:3;"></path></g><g data-mml-node="mo" transform="translate(4479.8,0)"><path data-c="2212" d="M84 237T84 250T98 270H679Q694 262 694 250T679 230H98Q84 237 84 250Z" style="stroke-width:3;"></path></g><g data-mml-node="mn" transform="translate(5480,0)"><path data-c="31" d="M213 578L200 573Q186 568 160 563T102 556H83V602H102Q149 604 189 617T245 641T273 663Q275 666 285 666Q294 666 302 660V361L303 61Q310 54 315 52T339 48T401 46H427V0H416Q395 3 257 3Q121 3 100 0H88V46H114Q136 46 152 46T177 47T193 50T201 52T207 57T213 61V578Z" style="stroke-width:3;"></path></g><g data-mml-node="mo" transform="translate(5980,0)"><path data-c="29" d="M60 749L64 750Q69 750 74 750H86L114 726Q208 641 251 514T294 250Q294 182 284 119T261 12T224 -76T186 -143T145 -194T113 -227T90 -246Q87 -249 86 -250H74Q66 -250 63 -250T58 -247T55 -238Q56 -237 66 -225Q221 -64 221 250T66 725Q56 737 55 738Q55 746 60 749Z" style="stroke-width:3;"></path></g><g data-mml-node="mo" transform="translate(6369,0)"><path data-c="28" d="M94 250Q94 319 104 381T127 488T164 576T202 643T244 695T277 729T302 750H315H319Q333 750 333 741Q333 738 316 720T275 667T226 581T184 443T167 250T184 58T225 -81T274 -167T316 -220T333 -241Q333 -250 318 -250H315H302L274 -226Q180 -141 137 -14T94 250Z" style="stroke-width:3;"></path></g><g data-mml-node="mi" transform="translate(6758,0)"><path data-c="1D45E" d="M33 157Q33 258 109 349T280 441Q340 441 372 389Q373 390 377 395T388 406T404 418Q438 442 450 442Q454 442 457 439T460 434Q460 425 391 149Q320 -135 320 -139Q320 -147 365 -148H390Q396 -156 396 -157T393 -175Q389 -188 383 -194H370Q339 -192 262 -192Q234 -192 211 -192T174 -192T157 -193Q143 -193 143 -185Q143 -182 145 -170Q149 -154 152 -151T172 -148Q220 -148 230 -141Q238 -136 258 -53T279 32Q279 33 272 29Q224 -10 172 -10Q117 -10 75 30T33 157ZM352 326Q329 405 277 405Q242 405 210 374T160 293Q131 214 119 129Q119 126 119 118T118 106Q118 61 136 44T179 26Q233 26 290 98L298 109L352 326Z" style="stroke-width:3;"></path></g><g data-mml-node="mo" transform="translate(7440.2,0)"><path data-c="2212" d="M84 237T84 250T98 270H679Q694 262 694 250T679 230H98Q84 237 84 250Z" style="stroke-width:3;"></path></g><g data-mml-node="mn" transform="translate(8440.4,0)"><path data-c="31" d="M213 578L200 573Q186 568 160 563T102 556H83V602H102Q149 604 189 617T245 641T273 663Q275 666 285 666Q294 666 302 660V361L303 61Q310 54 315 52T339 48T401 46H427V0H416Q395 3 257 3Q121 3 100 0H88V46H114Q136 46 152 46T177 47T193 50T201 52T207 57T213 61V578Z" style="stroke-width:3;"></path></g><g data-mml-node="mo" transform="translate(8940.4,0)"><path data-c="29" d="M60 749L64 750Q69 750 74 750H86L114 726Q208 641 251 514T294 250Q294 182 284 119T261 12T224 -76T186 -143T145 -194T113 -227T90 -246Q87 -249 86 -250H74Q66 -250 63 -250T58 -247T55 -238Q56 -237 66 -225Q221 -64 221 250T66 725Q56 737 55 738Q55 746 60 749Z" style="stroke-width:3;"></path></g></g></g></svg><mjx-assistive-mml unselectable="on" display="inline" style="top: 0px; left: 0px; clip: rect(1px, 1px, 1px, 1px); user-select: none; position: absolute; padding: 1px 0px 0px; border: 0px; display: block; width: auto; overflow: hidden;"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>φ</mi><mo stretchy="false">(</mo><mi>n</mi><mo stretchy="false">)</mo><mo>=</mo><mo stretchy="false">(</mo><mi>p</mi><mo>−</mo><mn>1</mn><mo stretchy="false">)</mo><mo stretchy="false">(</mo><mi>q</mi><mo>−</mo><mn>1</mn><mo stretchy="false">)</mo></math></mjx-assistive-mml></mjx-container>' raw-content>$\varphi(n)$</ElTooltip>
- $m$，明文，就是要保护的信息。
- $c$，密文，就是要破解的信息。

下面的公式会具体展现明文和密文的转化。

#### 2. 公式

RSA算法中有以下公式

1. 加密公式

$$ c \equiv m^{e} \pmod n $$

2. 解密公式

$$ m \equiv c^{d} \pmod n $$

3. $e$ 和 $d$ 的关系式

$$ d \equiv e^{-1} \pmod{\varphi} $$

4. $p$ 和 $q$ 是素数的情况下，$n$ 的欧拉函数

$$ \varphi = (p-1)(q-1) $$

5. $n$ 与 $p, q$ 的关系式

$$ n = p \times q $$

6. 大小关系

$$
\begin{array}{c}
m < n \\
n \approx \varphi
\end{array}
$$

<Container type='tip'>

入门后可以尝试推导一下上面的公式，入门前只需懂得如何在 Python 中实现上面公式即可。
</Container>

#### 3. 原理

- **解密原理**

  一般 RSA 算法加密会公开公钥 $n$ 和 $e$，信息传输难免会泄露密文 $c$，所以我们可以得到的一共有三个元素 $n, e, c$. 从上面的解密公式可以知道我们需要密钥因子 $d$，而计算 $d$ 需要 $e$ 和 $\varphi$，$e$ 已知，求未知的 $\varphi$ 又需要 $p$ 和 $q$，有关 $p$ 和 $q$ 的式子只有一个 $n = p \times q$.

  因此解密的关键是在得到 $p$ 和 $q$.

  ![解密原理](/assets/images/learn/crypto-decrypt-schematic.png)

- **难破解的原理**

  RSA 算法开创了现代密码并实现了非对称加密，其中难破解的原理基于[大整数分解](https://zh.wikipedia.org/wiki/%E6%95%B4%E6%95%B0%E5%88%86%E8%A7%A3)。

  计算 $n = p \times q$ 简单，但由 $n$ 分解出 $p, q$ 非常困难，这是 RSA 算法的基础。

  对于较小的整数，手算数 $12$ 的因子，很快可以列出 $1,2,3,4,6,12$，但一旦上升到很大的整数，例如 <code style="word-break: break-all;">6969872410035233098344189258766624225446081814953480897731644163180991292913719910322241873463164232700368119465476508174863062276659958418657253738005689</code>，就极难拆成两个数字。

- **解题的原理**

  上面说到，$n$ 很难分解，那我们是靠什么来破解的呢？

  答：靠一个个特殊的情况，有时候出题人会给出泄露的 $d$，或者 $p$ 和 $q$ 之间的关系让我们数学推导等等特殊情况。针对这一个个特殊情况，我们也有相应的攻击方法，例如：

  - 小明文攻击
  - 低指数暴力破解
  - Rabin 攻击与中国剩余定理的初始应用
  - 小明文攻击
  - 广播攻击
    - 低加密指数
    - 不互素的 CRT
  - 拓展欧拉定理
  - Rabin 进阶
  - dp & dq 泄露
  - $p-1$ 光滑
  - AMM
  - RSA 证书
    - 手动处理
    - 代码处理
  - 变种 RSA — SchmidtSamoa
  - 复数与 RSA

#### 4. 数学基础

RSA 中涉及到我们之前没有或者说不太重视的一些数学知识点，接下来我们一起来学习一下。

- **取模**

  之前提到的公式中四个有三个都出现了一个英文单词 mod，在数学中，这被称为「模」，是一种运算，即求余运算，编程语言中常以 `%` 展现。例如，$17 \div 5 = 3 \cdots 2$，则 `17 % 5 = 2`，或者记作 $17 \equiv 2 \pmod 5$. 很容易发现，$17 \pmod 5$ 的值域只有 $[0,5)$ 中的整数。

- **模运算**

  模运算和平时的运算基本上没有太大区别，只是在模运算中，我们人为的划定了一个整数集合 $[0, n)$，模运算结果都不会超过这个集合。模运算也可以实现等式左右两边交换，例如 $e$ 和 $d$ 的关系式也可以写成 $e\times d \equiv 1 \pmod n$.

  而运算符号加减乘除的除就有些变化了。上面的 $e^{-1}$ 可以写成 $\frac {1}{e}$，但在模运算中不能直接除，

  例如 $\frac {a+1}{b-1} \pmod n$ 要先求b的逆元 $(b-1)^{-1}$，再将这逆元 $(b-1)^{-1}$ 与 $(a+1)$ 相乘。

  在上面第三个公式 $e$ 和 $d$ 的关系式中有 $d^{-1}$，这涉及到模运算中求逆元，这点展开会涉及到其他数学知识，这里先放着不谈。只要记住在模运算中，没有除法，只有求逆元然后再相乘。具体的代码实现会在下面讲述。

- **海纳百川的 $\boldsymbol{k}$ 和无所不能的公因数**

  在上面的取模和模运算中，涉及到大量的 $\mathrm{mod}\ n$ 操作，如何转化成可以手算推理的东西呢？

  答：$k \times n$，其中 $k$ 是任意整数，例如 $12 \pmod 5 = 12 - 2 \times 5$，其中 $k=2$.

  我们之后会遇到一部分的数学推导题，题目会给出有关 $p$ 和 $q$ 的关系式。草稿纸上推理处理完关系式后，经常能得到一个有关 $p$ 或者 $q$ 的式子 $a \equiv b \pmod p$，此时我们只需要将这个式子转化一下
  $$a = b + k_1 p \Rightarrow k_1 p = a - b$$
  然后求 $a-b$ 和 $n$ 的公因数就可以得到 $p$ 了。

  因为 $n = p \times q$ 也可以看成 $n = k_2 p$，而一般 $k_1 \ne k_2$，所以就可以得到 $p$.

  ::: info
  具体情况可参见 [羊城杯 2021 Bigrsa 共享素数](https://www.cnblogs.com/Clair-is-com/p/16470121.html)。
  :::

#### 5. 计算机基础

思考这样一个问题：将字符串 `我喜欢密码学` 转化为整数。

这看起来有点不可思议，文字和数字怎么联系在一起。但计算机上就可以，计算机底层是一堆二进制，只有 `0` 和 `1`。要想在屏幕上展现文字，就需要**编码**。编码将数字按照一定的法则转换成对应的文字，常见的编码如 [ASCII 编码](https://www.runoob.com/w3cnote/ascii.html)。

通过一些编码方式，我们就能把想保护传输的信息转换成整数并一一对应，再通过 RSA 算法加密计算得到密文 $c$.
$$ c \equiv m^{e} \pmod n $$

#### 6. 代码实现

加密脚本不需要掌握熟背，只需知道每一行的意思即可；解密脚本需熟练掌握。

::: code-group

```python:line-numbers [RSA 加密脚本]
# 导入 Crypto.Util.number 模块中的所有函数
from Crypto.Util.number import *
# 导入 gmpy2 模块，用于高性能的数学运算
import gmpy2
# 从 secret 模块导入 flag，通常用于表示隐藏信息
from secret import flag

# 这是给出了 flag 的样式，并不是真正的 flag，但已经提供了一些提示，有些题目会根据 flag 头来破解
flag = b"NSCTF{******}"

# 生成两个 256 位的质数 p 和 q
p = getPrime(256)
q = getPrime(256)

# 计算 n，即 p 和 q 的乘积，用于 RSA 算法的模数
n = p * q
# 定义公钥指数 e，通常为 65537
e = 65537
# 将 flag 转换为长整数
m = bytes_to_long(flag)

# 使用 gmpy2 模块的 powmod 函数进行模幂运算，加密消息得到密文 c
c = gmpy2.powmod(m, e, n)

# 打印模数 n
print(f'n = {n}')
# 打印公钥指数 e
print(f'e = {e}')
# 打印加密后的密文 c
print(f'c = {c}')
# 打印质数 p（通常在 CTF 中不会直接给出）
print(f'p = {p}')
# 打印质数 q（通常在 CTF 中不会直接给出）
print(f'q = {q}')

# 以下是输出的结果，这些值通常在 CTF 题目中给出
# n = 4024941574680124502316363981547051098032677531528457166859670261861728313081282635664023890534034586556845494323497683923813915739234466472396261320600483
# e = 65537
# c = 226967182640114431119923862488626190608050511354278604627242247124377735518111678279381846350389469161980779137969837090666693533616114290831130137310320
# p = 62658315832909660478685872111870233686035497063073558738980225214351386198939
# q = 64236351092062515945998729497153532140067861836088195242257976217499252460697
```

```python:line-numbers [RSA 解密脚本]
# 导入 Crypto.Util.number 模块中的所有函数，用于处理数字和字节之间的转换等
from Crypto.Util.number import *


"""输入部分"""
# 已知的模数 n，用于 RSA 加密和解密
n = 4024941574680124502316363981547051098032677531528457166859670261861728313081282635664023890534034586556845494323497683923813915739234466472396261320600483
# 已知的公钥指数 e，通常为 65537
e = 65537
# 已知的密文 c，需要被解密
c = 226967182640114431119923862488626190608050511354278604627242247124377735518111678279381846350389469161980779137969837090666693533616114290831130137310320
# 已知的质数 p，用于计算私钥
p = 62658315832909660478685872111870233686035497063073558738980225214351386198939
# 已知的质数 q，用于计算私钥
q = 64236351092062515945998729497153532140067861836088195242257976217499252460697


"""处理部分"""
# 计算欧拉函数 phi(n)，用于RSA算法中的私钥计算
phi = (p-1)*(q-1)
# 计算私钥指数 d，即 e 在模 phi(n) 的逆元
d = inverse(e,phi)
# 使用私钥指数 d 解密密文 c，得到明文 m，具体就是 m = c ** d (modn)
m = pow(c, d, n)
# 将解密后的长整数 m 转换回字符串，得到原始的 flag 信息
flag = long_to_bytes(m)


"""输出部分"""
# 打印解密后的 flag 信息
print(flag)
```

:::

## 以后的路

入门 RSA 算法后，可以选择跟着 [NSSCTF 的工坊课程](https://www.nssctf.cn/explore/workshop/) 自主继续深造 RSA，在这基础上一起学习 AES 对称加密、圆锥曲线算法、LCG 流密码、格密码和 DSA 密码等等。

::: tip 写在最后
密码路上道阻且长，希望同学们能坚持下去，必定能见到密码学的彩虹！
:::
