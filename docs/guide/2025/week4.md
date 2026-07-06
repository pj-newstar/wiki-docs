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

# Week 4

能站在这里，已经证明了你的毅力。

## Pwn

### content

<Container type='info'>

本题考查简单的堆喷概念。
</Container>

尝试直接通过栈溢出覆盖返回地址会发现这是一个正常情况下的非法地址。

- 在开始做这个题之前可以简单了解一下堆喷的概念和利用方式，参考 [CVE-2025-22457](https://attackerkb.com/topics/0ybGQIkHzR/cve-2025-22457/rapid7-analysis?referrer=notificationEmail).
- 在字符集受限的情况下配合对堆布局的控制使得最终落到合法的范围之内。
- 溢出可能覆盖某个结构体成员或函数指针，而不是直接覆盖返回地址。观察目标二进制中是否有函数指针被调用的地方。

:::tip
你可能需要对 HTTP 协议有一定了解，这样可以辅助你更快地逆向交互格式，甚至跳过逆向交互格式这一流程。
:::

### calc_queen

为了做出此题，你需要非常熟悉函数的栈帧结构。

另外 Week3 题目 calc_meow 的预期解虽与本体利用手法不同，但是都考察了你对函数栈帧的理解。我认为在尝试本道题之前，先想出 calc_meow 的预期解会对解出本题有所帮助。

另外，相比于前几周的 pwn 题，本题的考点较为新颖，因此直接在网络上检索到同类型的题目几乎是不太可能的。本题的难度较大，感到没有头绪是很正常的事情，切莫灰心丧气！

:::tip

- C 函数的局部变量通常存储在哪里？C 函数局部变量的地址通常是根据哪个寄存器偏移得到的？
- 修改函数的栈底会对函数局部变量布局造成什么影响？
- 本题预期解无需 leak 任何地址。

:::

### fmt&got

<Container type='info'>

本题考查利用格式化字符串漏洞覆盖内存。
</Container>

相信做完 Week3 的「fmt&canary」之后，你对「格式化字符串漏洞」以及「如何利用格式化字符串漏洞来泄露内存」有了一定的了解，在开始做这道题之前，你可以再了解一下：

- <Link icon="external" theme="underline hover" href="https://ctf-wiki.org/pwn/linux/user-mode/fmtstr/fmtstr-exploit/#_8">覆盖内存</Link>
- <Link icon="external" theme="underline hover" href="https://bbs.kanxue.com/thread-282434.htm">Linux 程序保护机制</Link>

请善用你的 GDB.

## Web

### 小 E 的留言板

<Container type='info'>

本题考查基础 XSS 以及绕过。
</Container>

为了获取到小 E 浏览器内的 cookie，我们需要进行跨站脚本攻击<span data-desc>（XSS）</span>。一般 xss 需要逃逸出当前属性或标签，利用特殊标签<span data-desc>（例如 script）</span>或者属性<span data-desc>（例如 onerror，onfocus 等）</span>来执行 JavaScript。但并不是所有题目都能逃逸出标签从而引入恶意标签，因此可以考虑如何闭合属性并引入恶意属性来执行 JavaScript。

其次题目会有一些针对特殊字符或字符串的过滤，需要进行相应的绕过，常见的有大小写绕过，双写绕过等。

可以先本地测试，执行一个弹窗的 js，成功后改为发送 Cookie 到 [dnslog 平台](http://dnslog.cn/)。

你可以通过 Drunkbaby 师傅的 [从 0 到 1 完全掌握 XSS](https://drun1baby.top/2022/05/05/%E4%BB%8E0%E5%88%B01%E5%AE%8C%E5%85%A8%E6%8E%8C%E6%8F%A1XSS/) 来对 XSS 漏洞进行一个基础的学习。

更进阶的，你可以阅读 P 神的 [浅析白盒安全审计中的 XSS Fliter](https://www.leavesongs.com/PENETRATION/xss-fliter-bypass.html) 来进行进一步的学习。

### sqlupload

在一般的 SQLi 题目中，可能会出现 flag 不在数据库中的情况，这时你需要通过利用 SQL 中特殊的函数来进行注入。

在羊城杯 2025 中，有一题 [Ezsignin](https://www.blog.st4rr.top/writeups/%E7%BE%8A%E5%9F%8E%E6%9D%AF2025Writeup.pdf)，该题利用 SQLite 中的特性，注入恶意语句到数据库中，然后使用 ATTACH 语句将恶意代码注入到 `/app/views/upload.ejs` 来进行数据库层面之外的攻击<span data-desc>（这里指 SSTI）</span>。更多细节可以参见 [SQLite Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/d49faf9874bc964e855c2d2ce46764c0552fa99a/SQL%20Injection/SQLite%20Injection.md#attach-database).

本题具有类似的原理，但是实践方式更简单，你需要在特定的位置注入你的恶意代码，然后想办法将其保存在可被执行的位置。

本题根目录下的 `flag` 文件被保护了起来，你需要通过执行 `readflag` 来获得 FLAG.

::: tip
本题给予了完整的 Docker 环境附件，你可以在 Linux 上安装 Docker 环境来进行使用<span data-desc>（官方安装教程：[Install Docker](https://docs.docker.com/engine/install/ubuntu/)）</span>。
:::

### ssti 在哪里？

服务端请求伪造（Server-Side Request Forgery, SSRF），是指构造 Payload 让服务端去发送一个请求，由于服务端位于内网，往往能通过 SSRF 攻击内网靶标。漏洞的核心在于通过构造请求，使已被控制的服务器发送任意请求到后端内容。

如果你对 SSRF 完全不了解，你可以查看 Drunkbaby 师傅总结的 [从 0 到 1 完全掌握 SSRF](https://drun1baby.top/2022/05/16/从0到1完全掌握SSRF/) 来进行学习。
