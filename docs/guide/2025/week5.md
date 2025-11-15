---
titleTemplate: ":title | 参考文档 - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
import Link from '@/components/docs/Link.vue'
import { ElTag, ElCollapse, ElCollapseItem, ElTooltip } from 'element-plus'
import 'element-plus/es/components/tag/style/css'
import 'element-plus/es/components/collapse/style/css'
import 'element-plus/es/components/collapse-item/style/css'
import 'element-plus/es/components/tooltip/style/css'
</script>

# Week 5

本周的考点已经相当接近于主流 CTF 赛事中的常见内容。

## Pwn

### 复读机堂堂归来！复读机堂堂归来！

<Container type='info'>

本题考查利用非栈上格式化字符串漏洞泄露及覆盖内存。
</Container>

相信经过前两周的历练，你应该对格式化字符串漏洞利用比较熟悉了。对于常规的栈上格式化字符串漏洞，可以任意构造自己的恶意数据来实现任意地址写，但是对于非栈上变量来说，就无法直接给出目的地址的指针。此时你就需要留意栈上残留的内容，看看能不能找到可以利用的点。

还是这句话：请善用你的 GDB。

### stdout

<Container type='info'>

本题考查 `_IO_FILE` 结构体利用。
</Container>

在开始做这道题之前，你可以先了解一下 [FILE 结构](https://ctf-wiki.org/pwn/linux/user-mode/io-file/introduction/)。

对 FILE 结构体有了大概的认知之后就可以开始思考下面这些问题:

1. `printf` / `puts` 函数与 `vtable` 有什么关系。
2. 什么样的 `vtable` 才是合法的。
3. 怎么利用 `_wide_data` 来配合自己伪造的 `vtable`。

## Web

### 废弃的网站

<Container type='info'>

本题考查了「条件竞争」漏洞的简单利用。
</Container>

本题给出了服务的源码，你需要找出服务中存在的**条件竞争点**，并且通过条件竞争点来合理编写脚本来对服务进行攻击。

### 眼熟的计算器

<Container type='info'>

本题考查了选手对于 Java Spring 中的 jar 包进行分析的能力。
</Container>

在本题中，你需要学习如何将 Java Spring 题目中给出的 jar 包反编译成能够阅读的 java 伪代码。

推荐使用 JetBrains 公司的 [IDEA](https://www.jetbrains.com/zh-cn/idea/download/?section=windows) 来完成这个操作。

本题将会是你迈向 java 学习的一小步。

### 小 W 和小 K 的故事（最终章）

<Container type='info'>

本题考查了 nodejs 中的「原型链污染」漏洞。
</Container>

尝试找出服务中存在的各个模块中，可能造成原型链污染的方法，并且上网搜寻其相关内容。

当我们在对大多数服务<span data-desc>（特别是 go，nodejs，python 等强模块相关的语言）</span>进行渗透时，通常需要了解该服务中所利用到的模块的版本信息，并且根据模块版本信息来上网搜寻相关的 CVE，或者查询 github 上的历史 commit 中被修改掉的内容，很可能就是解题的关键。
