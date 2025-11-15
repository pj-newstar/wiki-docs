---
titleTemplate: ":title | 参考文档 - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# Week 1

每年的 Week 1 都是一场新的征途的起点。愿各位不畏艰险，砥砺前行。

如果你在做题时遇到了一些困难，可以阅读这里的内容，或许能够给你一些启发。

## Pwn

本周的 Pwn 考查一些工具的基本使用，可以阅读 [Pwn 环境配置](/learn/configure-pwn)，并熟悉其中提到的工具。

对于快速入门和解题样例，可以阅读 [Pwn 二进制安全 - 快速入门](/learn/pwn)。

## Reverse

### Puzzle

你需要学会如何熟练使用 IDA 中的分析工具辅助你进行程序分析，答案就藏在字里行间。

你应该了解如下有关 IDA 分析逆向程序的快捷键，例如：

- <kbd>⇧ Shift</kbd><kbd>F12</kbd> 查询程序中出现的字符串。
- 在选中或单击字符串，变量或者函数后，使用 <kbd>X</kbd> 键来查看其所被引用的位置。
- <kbd>⇧ Shift</kbd><kbd>E</kbd> 可以帮助你打开数据提取界面，将程序中的数据提取出来

IDA 将一直伴随逆向之旅，祝你玩得开心！

### plzdebugme

调试是逆向操作中辅助分析的极好方法。

逆向分析中动态分析有助于你在被混乱的静态代码中，理清程序执行的逻辑。或者提取一些在程序运行过程中才会生成的数据。

如果你想调试 ELF 文件，你可以利用 IDA 自带的 Remote GDB debugger 功能，连接到自己的 linux 虚拟机上进行调试。可参考 [IDA 远程动态调试](https://blog.csdn.net/m0_46296905/article/details/115794076)。

## Web

### 黑客小 W 的故事（1）

<Container type='tip'>

如果你完全不了解什么是 HTTP 协议，你可以查看 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP) 的有关内容，来了解更多。
</Container>

如果你了解空洞骑士，你应该知道骨钉大师一共有三位，对应题目的三个关卡。每一个关卡中都存在各种提示来帮助你完成这个题目。

本题考察了选手对于 Web 相关工具的使用以及对于 HTTP 协议的熟悉程度，下面这些问题对更好地解决问题或许会有帮助：

- 什么是抓包？我们可以怎么篡改数据包？
- 什么是 Method？关于不同的 Method 有什么特点？
- 关于请求时产生的各个相应头有什么区别？User-Agent 的标准格式是什么样子的？
- Cookie 是什么？有什么用处？

如果你不知道怎么去修改这些内容，你可以尝试使用 Yakit, BurpSuite, curl 等工具，或者使用浏览器插件 Hackbar 来完成。

<Container type='warning'>

本题设计 DELETE 方法的部分内容并不符合[协议标准](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/DELETE)。请大家在后续遇到 DELETE 方法时不要被本题误导。
</Container>

## Misc

### 前有文字，所以搜索很有用

如题所述，这道题解题的关键就在于「文字」与「搜索」，文字隐写是考点，隐写方法很多，期望你通过搜索和阅读前人的文字进行学习。

Track1 的考点为零宽字符隐写，Track2 为 brainfuck 编码与 SNOW 隐写，Track3 为字频统计。

如果你对这些知识点不太熟悉，你可以自行搜索或者查看 [Misc 指南](/learn/misc)。
