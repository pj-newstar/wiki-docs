---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 采一朵花，送给艾达（1）



程序主函数：

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  _main(argc, argv, envp);
  tutorial();
  puts("Now try to solve this problem.");
  JUMPOUT(0x140001788LL);
}
```

查看教程函数：

```cpp
void tutorial()
{
  puts("Junk codes are some assembly codes that can obfuscate your disassembler and decompiler.");
  puts("They usually use some small tricks to confuse disassembler or change the way of programs running.");
  puts("When disassembler try to disassemble the machine codes, it will not execute the code at the same time.");
  puts("It will only try to recognize all the data, code by code.");
  puts("Therefore, when some special codes exist but will not execute when running, they will confuse the disassembler.");
  puts(
    "For example, 0xE8(call), 0x74 & 0x75(jz&jnz) will make the disassembler consider that the data after these codes is "
    "the address to jump.");
  puts(
    "However, if these codes are skipped when running(junkcode), the disassembler will not know this. It will make mistakes.");
  puts("You can try to PATCH the program (for example NOP the junk codes) and make the real logic appear.");
  puts(
    "Here are some examples(you can try to patch them and if you can see the next example instead of JUMPOUT, that means you successed):");
  puts("1.use jz & jnz to replace jmp, and use 0xE8 to confuse the disassembler:");
  JUMPOUT(0x1400014F5LL);
}
```

实际上教程函数里面写的已经很明白了，花指令就是会干扰反编译器的垃圾字节码，在实际过程中会跳过一些字节不执行，去除花指令的核心方式就是 nop<span data-desc>（改为无操作）</span>。

第一个花指令：

```asm
.text:00000001400014F0                 jz      short near ptr loc_1400014F4+1
.text:00000001400014F2                 jnz     short near ptr loc_1400014F4+1
.text:00000001400014F4
.text:00000001400014F4 loc_1400014F4:                          ; CODE XREF: tutorial+A0↑j
.text:00000001400014F4                                         ; tutorial+A2↑j
.text:00000001400014F4                 call    near ptr 16405A241h
```

很明显这里是不管真假都跳转到 `0x001400014F4 + 1`<span data-desc>（即 `0x001400014F5`）</span>的位置，所以要先在 `0x001400014F4` 的位置按 <kbd>U</kbd> 取消定义，再在 `0x001400014F5` 位置按 <kbd>C</kbd>，后面的部分就可以继续识别了，然后可以把 `jz` + `jnz` + `0xE8` 给 nop 掉。

第二个花指令：

```asm
.text:00000001400014F5 loc_1400014F5:                          ; CODE XREF: tutorial+A0↑j
.text:00000001400014F5                                         ; tutorial+A2↑j
.text:00000001400014F5                 lea     rax, a2UseCallChange ; "2.use call, change esp and return to co"...
.text:00000001400014FC                 mov     rcx, rax        ; Buffer
.text:00000001400014FF                 call    puts
.text:0000000140001504                 call    loc_14000150A
.text:0000000140001504 ; ---------------------------------------------------------------------------
.text:0000000140001509                 db  83h
.text:000000014000150A ; ---------------------------------------------------------------------------
.text:000000014000150A
.text:000000014000150A loc_14000150A:                          ; CODE XREF: tutorial+B4↑j
.text:000000014000150A                 db      36h
.text:000000014000150A                 add     [rsp+20h+var_20], 8
.text:000000014000150F                 retn
.text:000000014000150F ; ---------------------------------------------------------------------------
.text:0000000140001510                 db 0F3h
.text:0000000140001511                 db  48h ; H
.text:0000000140001512                 db  8Dh
```

这里是一个修改了返回地址的花指令，call 了下面的代码后改了返回地址，经过计算可以得到返回到 `0x00140001511` 的位置，即使不计算也可以试试从 `0x00140001510` 开始往下按 <kbd>C</kbd>，按到 `0x00140001511` 的时候能成功识别，然后把 `0x00140001504` ~ `0x00140001510` 都 nop 掉即可

第三个花指令：

```asm
.text:0000000140001511                 lea     rax, a3MixedOrOther ; "3.mixed or other: "
.text:0000000140001518                 mov     rcx, rax        ; Buffer
.text:000000014000151B                 call    puts
.text:0000000140001520                 xor     eax, eax
.text:0000000140001522
.text:0000000140001522 loc_140001522:                          ; CODE XREF: tutorial+E1↓j
.text:0000000140001522                 jz      short loc_14000152F
.text:0000000140001524
.text:0000000140001524 loc_140001524:                          ; CODE XREF: tutorial:loc_140001524↑j
.text:0000000140001524                 jz      short near ptr loc_140001524+1
.text:0000000140001526                 jnz     short near ptr loc_140001528+1
.text:0000000140001528
.text:0000000140001528 loc_140001528:                          ; CODE XREF: tutorial+D6↑j
.text:0000000140001528                                         ; tutorial:loc_14000152F↓j
.text:0000000140001528                 call    near ptr 141E81DA1h
.text:000000014000152D                 retn
.text:000000014000152D ; ---------------------------------------------------------------------------
.text:000000014000152E                 db 0E8h
.text:000000014000152F ; ---------------------------------------------------------------------------
.text:000000014000152F
.text:000000014000152F loc_14000152F:                          ; CODE XREF: tutorial:loc_140001522↑j
.text:000000014000152F                 jz      short near ptr loc_140001528+1
.text:0000000140001531                 jnz     short near ptr loc_140001522+1
.text:0000000140001533                 nop
.text:0000000140001534                 add     rsp, 20h
.text:0000000140001538                 pop     rbp
.text:0000000140001539                 retn
```

这是一个很复杂的复合花指令，有很多跳转，但是和之前一样，一点一点分析其跳转到了什么地方，然后对有 +1 这种偏移量的位置按 <kbd>U</kbd> 取消定义，再在偏移量的地方重新按 <kbd>C</kbd><span data-desc>（比如 `loc_140001524 + 1`，那就在 `0x00140001524` 按 <kbd>U</kbd>，`0x00140001525` 按 <kbd>C</kbd>）</span>，之后把从 `0x00140001522` 的地方到 `0x00140001533` 的地方全部 nop 掉就行了，此时函数终于可以正常反编译：

```cpp
void tutorial()
{
  puts("Junk codes are some assembly codes that can obfuscate your disassembler and decompiler.");
  puts("They usually use some small tricks to confuse disassembler or change the way of programs running.");
  puts("When disassembler try to disassemble the machine codes, it will not execute the code at the same time.");
  puts("It will only try to recognize all the data, code by code.");
  puts("Therefore, when some special codes exist but will not execute when running, they will confuse the disassembler.");
  puts(
    "For example, 0xE8(call), 0x74 & 0x75(jz&jnz) will make the disassembler consider that the data after these codes is "
    "the address to jump.");
  puts(
    "However, if these codes are skipped when running(junkcode), the disassembler will not know this. It will make mistakes.");
  puts("You can try to PATCH the program (for example NOP the junk codes) and make the real logic appear.");
  puts(
    "Here are some examples(you can try to patch them and if you can see the next example instead of JUMPOUT, that means you successed):");
  puts("1.use jz & jnz to replace jmp, and use 0xE8 to confuse the disassembler:");
  puts("2.use call, change esp and return to confuse the disassembler:");
  puts("3.mixed or other: ");
}
```

此时再查看 `main`：

```asm
.text:0000000140001781                 xor     eax, eax
.text:0000000140001783                 jz      short near ptr loc_140001787+1
.text:0000000140001785                 jnz     short near ptr loc_140001787+1
.text:0000000140001787
.text:0000000140001787 loc_140001787:                          ; CODE XREF: main+2B↑j
.text:0000000140001787                                         ; main+2D↑j
.text:0000000140001787                 call    near ptr 14805A4D4h
.text:000000014000178C                 sub     eax, 89480000h
```

和之前的第一条方法一样，从 `0x00140001783` 到 `0x00140001787` nop <span data-desc>（`0x00140001788` 不 nop）</span>：

```asm
.text:0000000140001781                 xor     eax, eax
.text:0000000140001783                 nop
.text:0000000140001784                 nop
.text:0000000140001785                 nop
.text:0000000140001786                 nop
.text:0000000140001787                 nop
.text:0000000140001788                 lea     rax, aInputYourFlag ; "Input your flag: "
```

第二个花指令：

```asm
.text:0000000140001804 loc_140001804:                          ; CODE XREF: main+82↑j
.text:0000000140001804                 xor     eax, eax
.text:0000000140001806                 jz      short near ptr loc_14000180F+3
.text:0000000140001808                 jnz     short near ptr loc_14000180F+3
.text:000000014000180A                 call    near ptr 14874DB10h
.text:000000014000180F
.text:000000014000180F loc_14000180F:                          ; CODE XREF: main+AE↑j
.text:000000014000180F                                         ; main+B0↑j
.text:000000014000180F                 call    near ptr 13974DB15h
.text:0000000140001814                 call    near ptr 0F848DB1Ah
.text:0000000140001814 ; ---------------------------------------------------------------------------
.text:0000000140001819                 db 0C7h, 7Fh, 0C1h, 43h, 3, 64h, 75h
.text:0000000140001820 ; ---------------------------------------------------------------------------
.text:0000000140001820                 adc     [rax-77h], ecx
.text:0000000140001823                 mov     al, ds:0C0F6558CB888B848h
```

和之前的第三种一样，仔细追踪跳转并修改定义为代码的部分就行：

```asm
.text:0000000140001804 loc_140001804:                          ; CODE XREF: main+82↑j
.text:0000000140001804                 xor     eax, eax
.text:0000000140001806                 jz      short loc_140001812
.text:0000000140001808                 jnz     short loc_140001812
.text:0000000140001808 ; ---------------------------------------------------------------------------
.text:000000014000180A                 db 0E8h
.text:000000014000180B                 db    1
.text:000000014000180C                 db 0C3h
.text:000000014000180D ; ---------------------------------------------------------------------------
.text:000000014000180D
.text:000000014000180D loc_14000180D:                          ; CODE XREF: main:loc_140001812↓j
.text:000000014000180D                 jz      short loc_140001817
.text:000000014000180D ; ---------------------------------------------------------------------------
.text:000000014000180F                 db 0E8h
.text:0000000140001810                 db    1
.text:0000000140001811                 db 0C3h
.text:0000000140001812 ; ---------------------------------------------------------------------------
.text:0000000140001812
.text:0000000140001812 loc_140001812:                          ; CODE XREF: main+AE↑j
.text:0000000140001812                                         ; main+B0↑j
.text:0000000140001812                 jz      short loc_14000180D
.text:0000000140001812 ; ---------------------------------------------------------------------------
.text:0000000140001814                 db 0E8h
.text:0000000140001815                 db    1
.text:0000000140001816                 db 0C3h
.text:0000000140001817 ; ---------------------------------------------------------------------------
.text:0000000140001817
.text:0000000140001817 loc_140001817:                          ; CODE XREF: main:loc_14000180D↑j
.text:0000000140001817                 mov     rax, 1175640343C17FC7h
```

之后把这些跳转全部 nop 掉，反编译：

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  _QWORD v4[6]; // [rsp+20h] [rbp-60h]
  _BYTE v5[256]; // [rsp+50h] [rbp-30h] BYREF
  _BYTE v6[1036]; // [rsp+150h] [rbp+D0h] BYREF
  unsigned int v7; // [rsp+55Ch] [rbp+4DCh]
  char *v8; // [rsp+560h] [rbp+4E0h]
  unsigned int v9; // [rsp+56Ch] [rbp+4ECh]
  char *Str; // [rsp+570h] [rbp+4F0h]
  int i; // [rsp+57Ch] [rbp+4FCh]

  _main(argc, argv, envp);
  tutorial();
  puts("Now try to solve this problem.");
  printf("Input your flag: ");
  Str = (char *)malloc(0x64u);
  scanf("%s", Str);
  v9 = strlen(Str);
  if ( v9 == 40 )
  {
    v4[0] = 0x1175640343C17FC7LL;
    v4[1] = 0xDF23C0F6558CB888uLL;
    v4[2] = 0xF2F082F69E2E0F4DuLL;
    v4[3] = 0xE1278329086B51BCuLL;
    v4[4] = 0x4E4F80B188C6BDCBLL;
    v8 = "EasyJunkCodes";
    v7 = strlen("EasyJunkCodes");
    rc4_init(v5, "EasyJunkCodes", v7);
    memcpy(v6, Str, (int)v9);
    rc4_crypt(v5, v6, v9);
    for ( i = 0; i < (int)v9; ++i )
    {
      if ( v6[i] != *((_BYTE *)v4 + i) )
      {
        printf("Wrong flag!");
        free(Str);
        return 0;
      }
    }
    printf("Right flag!");
    free(Str);
    return 0;
  }
  else
  {
    printf("Wrong length!");
    free(Str);
    return 114514;
  }
}
```

可以看到密文 `v4`，密钥 `v8`，加密算法 RC4，查看 `RC4_init`：

```cpp
void __fastcall rc4_init(__int64 a1)
{
  int i; // [rsp+Ch] [rbp-4h]

  for ( i = 0; i <= 255; ++i )
    *(_BYTE *)(i + a1) = -(char)i;
  JUMPOUT(0x140001589LL);
}
```

有花，继续去花

```cpp
__int64 __fastcall rc4_init(unsigned __int8 *a1, unsigned __int8 *a2, int a3)
{
  __int64 result; // rax
  unsigned __int8 v4; // [rsp+7h] [rbp-9h]
  int v5; // [rsp+8h] [rbp-8h]
  int i; // [rsp+Ch] [rbp-4h]
  int j; // [rsp+Ch] [rbp-4h]

  v5 = 0;
  for ( i = 0; i <= 255; ++i )
    a1[i] = -(char)i;
  result = 0;
  for ( j = 0; j <= 255; ++j )
  {
    v5 = (a1[j] + v5 + a2[j % a3]) % 256;
    v4 = a1[j];
    a1[j] = a1[v5];
    result = v4;
    a1[v5] = v4;
  }
  return result;
}
```

可以看到 S 盒初始化的地方被魔改，继续查看 PRGA：

```cpp
__int64 __fastcall rc4_crypt(__int64 a1, __int64 a2, int a3)
{
  __int64 result; // rax
  char v4; // [rsp+3h] [rbp-Dh]
  int v5; // [rsp+8h] [rbp-8h]

  result = 0;
  if ( a3 > 0 )
  {
    v5 = *(unsigned __int8 *)(a1 + 1) % 256;
    v4 = *(_BYTE *)(a1 + 1);
    *(_BYTE *)(a1 + 1) = *(_BYTE *)(a1 + v5);
    *(_BYTE *)(v5 + a1) = v4;
    JUMPOUT(0x140001710LL);
  }
  return result;
}
```

也有花，继续去花。

```cpp
__int64 __fastcall rc4_crypt(unsigned __int8 *a1, unsigned __int8 *a2, int a3)
{
  __int64 result; // rax
  unsigned __int8 v4; // [rsp+3h] [rbp-Dh]
  int i; // [rsp+4h] [rbp-Ch]
  int v6; // [rsp+8h] [rbp-8h]
  int v7; // [rsp+Ch] [rbp-4h]

  v7 = 0;
  v6 = 0;
  for ( i = 0; ; ++i )
  {
    result = (unsigned int)i;
    if ( i >= a3 )
      break;
    v7 = (v7 + 1) % 256;
    v6 = (a1[v7] + v6) % 256;
    v4 = a1[v7];
    a1[v7] = a1[v6];
    a1[v6] = v4;
    a2[i] += a1[(unsigned __int8)(a1[v7] + a1[v6])];
  }
  return result;
}
```

可以发现尾部改成了加号，一共只有这两个魔改点，提取密文密钥编写脚本同构：

```py
def rc4_init(key):
    S = list(range(256))
    for i in range(256):
        S[i] = (256 - i) & 0xFF
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    return S

def rc4_encrypt_decrypt(data, key):
    S = rc4_init(key)
    i = 0
    j = 0
    result = bytearray()
    for byte in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        K = S[(S[i] + S[j]) % 256]
        result.append((byte + K)&0xFF)
    return bytes(result)
```

把结尾的加号改为减号即可实现解密。

```py
def rc4_init(key):
    S = list(range(256))
    for i in range(256):
        S[i] = (256 - i) & 0xFF
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    return S

def rc4_encrypt_decrypt(data, key):
    S = rc4_init(key)
    i = 0
    j = 0
    result = bytearray()
    for byte in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        K = S[(S[i] + S[j]) % 256]
        result.append((byte - K)&0xFF)
    return bytes(result)

plaintext = bytearray.fromhex("c77fc1430364751188b88c55f6c023df4d0f2e9ef682f0f2bc516b08298327e1cbbdc688b1804f4e")
key = b"EasyJunkCodes"
ciphertext = rc4_encrypt_decrypt(plaintext, key)
print(ciphertext)
```

最后的 flag 是 `flag{Junk_C0d3s_4Re_345y_t0_rEc0gn1Ze!!}`。