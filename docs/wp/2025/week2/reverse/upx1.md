---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 尤皮·埃克斯历险记（1）

将 exe 拖入 IDA 之后发现主函数不见了，程序只有一两个函数，再用 DIE 或者 exeinfope 查壳，发现有 UPX 壳。

`upx -d` 可以脱壳。

脱壳后查看主函数：

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  __int64 n34_1; // rax
  std::ostream *v4; // rax
  unsigned __int64 v5; // rax
  std::ostream *v6; // rax
  _BYTE v8[32]; // [rsp+20h] [rbp-60h] BYREF
  _BYTE v9[2]; // [rsp+40h] [rbp-40h] BYREF
  _BYTE v10[16]; // [rsp+50h] [rbp-30h] BYREF
  _BYTE v11[22]; // [rsp+60h] [rbp-20h] BYREF
  char v12; // [rsp+76h] [rbp-Ah]
  char v13; // [rsp+77h] [rbp-9h]
  __int64 n34; // [rsp+78h] [rbp-8h]
  unsigned __int64 i; // [rsp+80h] [rbp+0h]
  char v16; // [rsp+8Fh] [rbp+Fh]

  _main();
  qmemcpy(v8, "isfhGJ\tt~cU\ny\nuTjcj\tT~cjQdu~w{", 30);
  v8[30] = 4;
  v8[31] = 5;
  qmemcpy(v9, "qA", sizeof(v9));
  n34 = 34;
  std::string::string((std::string *)v11);
  std::operator<<<std::char_traits<char>>(refptr__ZSt4cout, "Enter your flag: ");
  std::operator>><char>(refptr__ZSt3cin, (std::string *)v11);
  encrypt((const std::string *)v10);
  n34_1 = std::string::length((std::string *)v10);
  if ( n34_1 == n34 )
  {
    v16 = 1;
    for ( i = 0; ; ++i )
    {
      v5 = std::string::length((std::string *)v10);
      if ( v5 <= i )
        break;
      if ( IsDebuggerPresent() )
      {
        v12 = *(_BYTE *)std::string::operator[](v10, i) ^ 0xC3;
        if ( v8[i] != v12 )
        {
          v16 = 0;
          break;
        }
      }
      else
      {
        v13 = *(_BYTE *)std::string::operator[](v10, i) ^ 0x3C;
        if ( v8[i] != v13 )
        {
          v16 = 0;
          break;
        }
      }
    }
    if ( v16 )
      v6 = (std::ostream *)std::operator<<<std::char_traits<char>>(refptr__ZSt4cout, "Right!");
    else
      v6 = (std::ostream *)std::operator<<<std::char_traits<char>>(refptr__ZSt4cout, "Wrong!");
    refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_(v6);
  }
  else
  {
    v4 = (std::ostream *)std::operator<<<std::char_traits<char>>(refptr__ZSt4cout, "Wrong!");
    refptr__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_(v4);
  }
  std::string::~string((std::string *)v10);
  std::string::~string((std::string *)v11);
  return 0;
}
```

发现给出了密文，还有一个 `encrypt` 函数：

```cpp
__int64 __fastcall encrypt(__int64 a1, __int64 a2)
{
  unsigned __int64 i_1; // rax
  char v4; // [rsp+27h] [rbp-19h] BYREF
  char *v5; // [rsp+28h] [rbp-18h]
  char C; // [rsp+37h] [rbp-9h]
  unsigned __int64 i; // [rsp+38h] [rbp-8h]

  v5 = &v4;
  std::string::basic_string<std::allocator<char>>(a1, &unk_1400C7000, &v4);
  std::__new_allocator<char>::~__new_allocator(&v4);
  for ( i = 0; ; ++i )
  {
    i_1 = std::string::length(a2);
    if ( i >= i_1 )
      break;
    C = *(_BYTE *)std::string::operator[](a2, i);
    if ( (unsigned int)(C - 48) > 9 )
    {
      if ( islower(C) || isupper(C) )
        std::string::operator+=(a1, (unsigned int)(char)(0xBB - C));
      else
        std::string::operator+=(a1, (unsigned int)C);
    }
    else
    {
      std::string::operator+=(a1, (unsigned int)(char)(0x69 - C));
    }
  }
  return a1;
}
```

是一个凯撒密码的变体：

1. 如果字符是数字，那么反转其在 0~9 之间的顺序（也就是 `0x69-C`），0 ↔ 9，1 ↔ 8，...

2. 如果是字母，反转其在字母表中的顺序并切换大小写（也就是 `0xBB-C`），A ↔ z，b ↔ Y，...

而主函数后面逻辑中的 `IsDebuggerPresent` 函数是**反调试**函数，当存在调试器时返回真，否则为假。

由于题目正常运行时肯定不存在调试器，所以这里一定会按假的分支走<span data-desc>（也就是异或 `0x3C`）</span>，接下来动态调试，在 `std::string::string((std::string *)v11);` 这一行下断点，提取前面的密文`69736668474A09747E63550A790A75546A636A09547E636A5164757E777B04057141`。

在 cyberchef fromhex 后异或 `0x3C` 后得到 `UOZT{v5HB_i6E6IhV_V5hB_VmXIBKG89M}`，然后编写一个简单的 python 脚本即可恢复 flag：

```python
def func(s: str) -> str:
    result = []
    for c in s:
        if '0' <= c <= '9':
            transformed = chr(ord('9') - (ord(c) - ord('0')))
            result.append(transformed)
        elif c.isalpha():
            transformed = chr(0xBB - ord(c))
            result.append(transformed)
        else:
            result.append(c)
    return ''.join(result)

original = "UOZT{v5HB_i6E6IhV_V5hB_VmXIBKG89M}"
dec = func(original)
print(dec)
```

flag 为：`flag{E4sy_R3v3rSe_e4Sy_eNcrypt10n}`。