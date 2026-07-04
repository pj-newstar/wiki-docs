---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 采一朵花，送给艾达（2）

程序主函数中有花：

```asm
.text:000000014000198F                 call    free
.text:0000000140001994                 mov     eax, 1BF52h
.text:0000000140001999
.text:0000000140001999 loc_140001999:                          ; CODE XREF: main+9C↓j
.text:0000000140001999                 jmp     loc_140001AF0
.text:000000014000199E loc_14000199E:                          ; CODE XREF: main+67↑j
.text:000000014000199E                 xor     eax, eax
.text:00000001400019A0                 jz      short near ptr loc_1400019A4+1
.text:00000001400019A2                 jnz     short near ptr loc_1400019A4+1
.text:00000001400019A4
.text:00000001400019A4 loc_1400019A4:                          ; CODE XREF: main+93↑j
.text:00000001400019A4                                         ; main+95↑j
.text:00000001400019A4                 call    near ptr 0CA5CD1F1h
.text:00000001400019A9                 loope   near ptr loc_140001999+1
.text:00000001400019AB                 lock cmp ah, [rdi+48h]
.text:00000001400019B0                 mov     [rbp+500h+var_560], eax
.text:00000001400019B3                 mov     rax, 0DCBAFF24A65D456h
```

A4位置取消定义A5位置重定义，反编译：

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  _main(*(_QWORD *)&argc, argv, envp);
  printf(&Format);
  *(_QWORD *)&Size[1] = malloc(0x10000u);
  scanf("%s", *(_QWORD *)&Size[1]);
  Size[0] = strlen(*(const char **)&Size[1]);
  if ( Size[0] == 40 )
  {
    v4[0] = 0x673AF04AEFE18A5CLL;
    v4[1] = 0xDCBAFF24A65D456LL;
    v4[2] = 0xCFD8B1539076E546uLL;
    v4[3] = 0xFD469B79EF8A33B7uLL;
    v4[4] = 0x5985D24E20980BECLL;
    v8 = (char *)KeyGen();
    v7 = strlen(v8);
    rc4_init(v5, v8, v7);
    memcpy(v6, *(const void **)&Size[1], Size[0]);
    rc4_crypt(v5, v6, Size[0]);
    for ( i = 0; i < Size[0]; ++i )
    {
      if ( v6[i] != *((_BYTE *)v4 + i) )
      {
        printf(&Format_);
        free(*(void **)&Size[1]);
        return 0;
      }
    }
    printf(&Format__0);
    free(*(void **)&Size[1]);
    return 0;
  }
  else
  {
    puts(&Buffer);
    free(*(void **)&Size[1]);
    return 114514;
  }
}
```

Keygen中有类似的花，去除后反编译：

```cpp
const char *KeyGen()
{
  malloc(0x11u);
  return "PickingUpFlowers";
}
```

查看RC4init：

```cpp
__int64 rc4_init()
{
  v6 = 0;
  strcpy(Destination, "Wow! Why this function looks empty???");
  v4 = 0;
  memset(buf_, 0, sizeof(buf_));
  strcpy(Source, "Not all kinds of the junk codes will make IDA fail analyze and dispaly JUMPOUT.");
  strcpy(
    Try_to_use_TAB_to_switch_to_the_assembly_page_and_find_out_what,
    "Try to use TAB to switch to the assembly page and find out what happened to this function.");
  strcat(Destination, Source);
  strcat(Destination, Try_to_use_TAB_to_switch_to_the_assembly_page_and_find_out_what);
  return 0;
}
```

发现函数几乎是空的，没有重要逻辑，也没有jumpout，但是提示要查看汇编：

```asm
.text:0000000140001621                 call    strcat
.text:0000000140001626                 push    rax
.text:0000000140001627                 xor     rax, rax
.text:000000014000162A                 xor     rax, 17h
.text:000000014000162E                 sub     rax, 10h
.text:0000000140001632                 call    $+5
.text:0000000140001637
.text:0000000140001637 label_of_flower:
.text:0000000140001637                 add     rax, 0Dh
.text:000000014000163B                 xor     rax, 0
.text:000000014000163F                 and     rax, 7
.text:0000000140001643                 and     rax, 21h
.text:0000000140001647                 cmp     rax, 1
.text:000000014000164B                 jz      short label_continue_to_normal_code
.text:000000014000164D                 retn
.text:000000014000164E ; ---------------------------------------------------------------------------
.text:000000014000164E
.text:000000014000164E label_continue_to_normal_code:          ; CODE XREF: rc4_init+1FB↑j
.text:000000014000164E                 pop     rax
.text:000000014000164F                 mov     [rbp+170h+var_14], 0
.text:0000000140001659                 jmp     short loc_140001683
```

发现了call $+5和retn，符号表也给出了这里正常代码的恢复位置，将call $+5到retn的位置全部nop即可

```cpp
char *__fastcall rc4_init(unsigned __int8 *a1, unsigned __int8 *a2, int a3)
{
  v10 = 0;
  strcpy(Destination, "Wow! Why this function looks empty???");
  v7 = 0;
  memset(buf_, 0, sizeof(buf_));
  strcpy(Source, "Not all kinds of the junk codes will make IDA fail analyze and dispaly JUMPOUT.");
  strcpy(
    Try_to_use_TAB_to_switch_to_the_assembly_page_and_find_out_what,
    "Try to use TAB to switch to the assembly page and find out what happened to this function.");
  strcat(Destination, Source);
  result = strcat(Destination, Try_to_use_TAB_to_switch_to_the_assembly_page_and_find_out_what);
  for ( i = 0; i <= 255; ++i )
  {
    result = (char *)&a1[i];
    *result = i ^ 0xCC;
  }
  for ( i = 0; i <= 255; ++i )
  {
    v10 = (a1[i] + v10 + a2[i % a3]) % 256;
    v9 = a1[i];
    a1[i] = a1[v10];
    result = (char *)v9;
    a1[v10] = v9;
  }
  return result;
}
```

发现和正常的RC4_KSA相比略有魔改，PRGA函数也有类似之前的两个花，去除即可：

```cpp
void __fastcall rc4_crypt(unsigned __int8 *a1, _BYTE *a2, __int64 Size)
{
  v7 = 0;
  v6 = 0;
  for ( Size_1 = 0; Size_1 < (int)Size; ++Size_1 )
  {
    v7 = (v7 + 1) % 256;
    v6 = (a1[v7] + v6) % 256;
    v3 = a1[v7];
    a1[v7] = a1[v6];
    a1[v6] = v3;
    a2[Size_1] ^= a1[(unsigned __int8)(a1[v7] + a1[v6])];
  }
  for ( Size_2 = 0; Size_2 < (int)Size; ++Size_2 )
    a2[Size_2] += Size_2;
}
```

尾部多了一个加索引，解密时前置即可

编写解密脚本：

```py
def rc4(key, ciphertext):
    temp = bytearray()
    for i, b in enumerate(ciphertext):
        temp.append((b - i) % 256)
    S = list(range(256))
    for i in range(256):
        S[i] ^= 0xCC
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    i = j = 0
    plaintext = bytearray()
    for char in temp:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        t = (S[i] + S[j]) % 256
        plaintext.append(char ^ S[t])
    return bytes(plaintext)

enc = bytearray.fromhex("5c8ae1ef4af03a6756d4654af2afcb0d46e5769053b1d8cfb7338aef799b46fdec0b98204ed28559")
key = b"PickingUpFlowers"
dec = rc4(key, enc)
print(dec)
```

`flag{WO0o0O0w_So0Oo0o_m4Ny_F1oO0o0oW3R5}`
