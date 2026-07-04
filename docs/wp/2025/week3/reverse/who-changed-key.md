---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 谁改了我的密钥啊

## 分析

和 `Week2` 安卓一样，加密在 `so` 文件里

```cpp
__int64 __fastcall Java_work_pangbai_changemykey_FirstFragment_checkFlag(__int64 a1, __int64 a2, __int64 a3)
{
  const char *src; // x20
  int8x16_t v5; // [xsp+0h] [xbp-C0h] BYREF
  unsigned int v6[4]; // [xsp+10h] [xbp-B0h] BYREF
  __int128 v7; // [xsp+20h] [xbp-A0h]
  __int128 v8; // [xsp+30h] [xbp-90h]
  __int128 v9; // [xsp+40h] [xbp-80h]
  __int128 v10; // [xsp+50h] [xbp-70h]
  __int128 v11; // [xsp+60h] [xbp-60h]
  __int128 v12; // [xsp+70h] [xbp-50h]
  __int128 v13; // [xsp+80h] [xbp-40h]
  char dest[16]; // [xsp+90h] [xbp-30h] BYREF
  __int128 v15; // [xsp+A0h] [xbp-20h]
  __int64 v16; // [xsp+B8h] [xbp-8h]

  v16 = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  src = (const char *)(*(__int64 (__fastcall **)(__int64, __int64, _QWORD))(*(_QWORD *)a1 + 1352LL))(a1, a3, 0LL);
  __android_log_print(4, "native", "input:%s", src);
  *(_OWORD *)dest = 0u;
  v15 = 0u;
  strncpy(dest, src, 0x10uLL);
  v12 = 0u;
  v13 = 0u;
  v10 = 0u;
  v11 = 0u;
  v8 = 0u;
  v9 = 0u;
  *(_OWORD *)v6 = 0u;
  v7 = 0u;
  __android_log_print(4, "native", aKey, (unsigned __int8)MK);
  __android_log_print(4, "native", aKey, BYTE1(MK));
  __android_log_print(4, "native", aKey, BYTE2(MK));
  __android_log_print(4, "native", aKey, BYTE3(MK));
  __android_log_print(4, "native", aKey, BYTE4(MK));
  __android_log_print(4, "native", aKey, BYTE5(MK));
  __android_log_print(4, "native", aKey, BYTE6(MK));
  __android_log_print(4, "native", aKey, BYTE7(MK));
  __android_log_print(4, "native", aKey, BYTE8(MK));
  __android_log_print(4, "native", aKey, BYTE9(MK));
  __android_log_print(4, "native", aKey, BYTE10(MK));
  __android_log_print(4, "native", aKey, BYTE11(MK));
  __android_log_print(4, "native", aKey, BYTE12(MK));
  __android_log_print(4, "native", aKey, BYTE13(MK));
  __android_log_print(4, "native", aKey, BYTE14(MK));
  __android_log_print(4, "native", aKey, HIBYTE(MK));
  v5 = veorq_s8((int8x16_t)MK, (int8x16_t)xmmword_900);
  extendSecond(v6, (unsigned int *)&v5);
  iterate32((unsigned int *)dest, v6);
  if ( *(_DWORD *)&dest[12] == mm )
    return (*(_DWORD *)&dest[8] == dword_398C && *(_DWORD *)&dest[4] == unk_3990) & (unsigned __int8)(*(_DWORD *)dest == dword_3994);
  else
    return 0LL;
}
```

由于 `apk` 是 `release` 编译的，所以代码有很多优化，可能有点看不出原样。看到函数列表的 `encryptSM4` 应该也能猜到这题是考查国密 `SM4`。`MK` 其实就是密钥 `{1, 0, 0, 0, 1, 0, 0, 0, 4, 0, 0, 0, 5, 0, 0, 0}`，`mm` 和之后的那几个值是密文。

```cpp
.data:0000000000003988 B3 E9 0D B6 mm              DCD 0xB60DE9B3          ; DATA XREF: LOAD:0000000000000508↑o
.data:0000000000003988                                                     ; Java_work_pangbai_changemykey_FirstFragment_checkFlag+20C↑r ...
.data:000000000000398C 4C 7B 7A 7A dword_398C      DCD 0x7A7A7B4C          ; DATA XREF: Java_work_pangbai_changemykey_FirstFragment_checkFlag+218↑r
.data:0000000000003990 89 37 D0 C7 dword_3990      DCD 0xC7D03789          ; DATA XREF: Java_work_pangbai_changemykey_FirstFragment_checkFlag+218↑r
.data:0000000000003994 6C 8E 27 2A dword_3994      DCD 0x2A278E6C          ; DATA XREF: Java_work_pangbai_changemykey_FirstFragment_checkFlag+224↑r
.data:0000000000003994             ; .data         ends
```

没有其他的修改，但是网上的 `SM4` 解密网站其实是解不出来的，原因出在端序转化上，注意明文和密钥传入再到加密的数据在内存中是怎么样的。

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define SAR(x, n) (((x >> (32 - n))) | (x << n))

#define L1(BB) BB ^ SAR(BB, 2) ^ SAR(BB, 10) ^ SAR(BB, 18) ^ SAR(BB, 24)

#define L2(BB) BB ^ SAR(BB, 13) ^ SAR(BB, 23)

/*系统参数*/
unsigned int FK[4] = {0xa3b1bac6, 0x56aa3350, 0x677d9197, 0xb27022dc};

/*固定参数*/
unsigned int CK[32] =
    {
        0x00070e15, 0x1c232a31, 0x383f464d, 0x545b6269,
        0x70777e85, 0x8c939aa1, 0xa8afb6bd, 0xc4cbd2d9,
        0xe0e7eef5, 0xfc030a11, 0x181f262d, 0x343b4249,
        0x50575e65, 0x6c737a81, 0x888f969d, 0xa4abb2b9,
        0xc0c7ced5, 0xdce3eaf1, 0xf8ff060d, 0x141b2229,
        0x30373e45, 0x4c535a61, 0x686f767d, 0x848b9299,
        0xa0a7aeb5, 0xbcc3cad1, 0xd8dfe6ed, 0xf4fb0209,
        0x10171e25, 0x2c333a41, 0x484f565d, 0x646b7279};

/*τ变换 S 盒*/
unsigned char TAO[16][16] =
    {
        {0xd6, 0x90, 0xe9, 0xfe, 0xcc, 0xe1, 0x3d, 0xb7, 0x16, 0xb6, 0x14, 0xc2, 0x28, 0xfb, 0x2c, 0x05},
        {0x2b, 0x67, 0x9a, 0x76, 0x2a, 0xbe, 0x04, 0xc3, 0xaa, 0x44, 0x13, 0x26, 0x49, 0x86, 0x06, 0x99},
        {0x9c, 0x42, 0x50, 0xf4, 0x91, 0xef, 0x98, 0x7a, 0x33, 0x54, 0x0b, 0x43, 0xed, 0xcf, 0xac, 0x62},
        {0xe4, 0xb3, 0x1c, 0xa9, 0xc9, 0x08, 0xe8, 0x95, 0x80, 0xdf, 0x94, 0xfa, 0x75, 0x8f, 0x3f, 0xa6},
        {0x47, 0x07, 0xa7, 0xfc, 0xf3, 0x73, 0x17, 0xba, 0x83, 0x59, 0x3c, 0x19, 0xe6, 0x85, 0x4f, 0xa8},
        {0x68, 0x6b, 0x81, 0xb2, 0x71, 0x64, 0xda, 0x8b, 0xf8, 0xeb, 0x0f, 0x4b, 0x70, 0x56, 0x9d, 0x35},
        {0x1e, 0x24, 0x0e, 0x5e, 0x63, 0x58, 0xd1, 0xa2, 0x25, 0x22, 0x7c, 0x3b, 0x01, 0x21, 0x78, 0x87},
        {0xd4, 0x00, 0x46, 0x57, 0x9f, 0xd3, 0x27, 0x52, 0x4c, 0x36, 0x02, 0xe7, 0xa0, 0xc4, 0xc8, 0x9e},
        {0xea, 0xbf, 0x8a, 0xd2, 0x40, 0xc7, 0x38, 0xb5, 0xa3, 0xf7, 0xf2, 0xce, 0xf9, 0x61, 0x15, 0xa1},
        {0xe0, 0xae, 0x5d, 0xa4, 0x9b, 0x34, 0x1a, 0x55, 0xad, 0x93, 0x32, 0x30, 0xf5, 0x8c, 0xb1, 0xe3},
        {0x1d, 0xf6, 0xe2, 0x2e, 0x82, 0x66, 0xca, 0x60, 0xc0, 0x29, 0x23, 0xab, 0x0d, 0x53, 0x4e, 0x6f},
        {0xd5, 0xdb, 0x37, 0x45, 0xde, 0xfd, 0x8e, 0x2f, 0x03, 0xff, 0x6a, 0x72, 0x6d, 0x6c, 0x5b, 0x51},
        {0x8d, 0x1b, 0xaf, 0x92, 0xbb, 0xdd, 0xbc, 0x7f, 0x11, 0xd9, 0x5c, 0x41, 0x1f, 0x10, 0x5a, 0xd8},
        {0x0a, 0xc1, 0x31, 0x88, 0xa5, 0xcd, 0x7b, 0xbd, 0x2d, 0x74, 0xd0, 0x12, 0xb8, 0xe5, 0xb4, 0xb0},
        {0x89, 0x69, 0x97, 0x4a, 0x0c, 0x96, 0x77, 0x7e, 0x65, 0xb9, 0xf1, 0x09, 0xc5, 0x6e, 0xc6, 0x84},
        {0x18, 0xf0, 0x7d, 0xec, 0x3a, 0xdc, 0x4d, 0x20, 0x79, 0xee, 0x5f, 0x3e, 0xd7, 0xcb, 0x39, 0x48}};

unsigned int RK[32];
int sm4_to_tao(unsigned char *in, unsigned char *out, int len)
{
    unsigned char a, b, c, d;
    int i = 0;

    for (i = 0; i < len; i++)
    {
        a = in[i];
        b = a >> 4;
        c = a & 0x0f;
        d = TAO[b][c];
        out[i] = d;
    }

    return 0;
}

int sm4_to_xor_2(unsigned char *a, unsigned char *b, unsigned char *out, int len)
{
    int i = 0;

    for (i = 0; i < len; i++)
    {
        out[i] = a[i] ^ b[i];
    }

    return 0;
}

int sm4_to_xor_4(unsigned char *a, unsigned char *b, unsigned char *c, unsigned char *d, unsigned char *out, int len)
{
    int i = 0;

    for (i = 0; i < len; i++)
    {
        out[i] = a[i] ^ b[i] ^ c[i] ^ d[i];
    }

    return 0;
}
int sm4_get_rki(unsigned int *mk)
{
    unsigned int k[36];
    unsigned int u, v, w;
    int i = 0;
    int j = 0;

    sm4_to_xor_2((unsigned char *)mk, (unsigned char *)FK, (unsigned char *)k, 16);

    for (i = 0; i < 32; i++)
    {
        sm4_to_xor_4((unsigned char *)(k + i + 1), (unsigned char *)(k + i + 2), (unsigned char *)(k + i + 3), (unsigned char *)(CK + i), (unsigned char *)(&u), 4);

        sm4_to_tao((unsigned char *)(&u), (unsigned char *)(&v), 4);

        w = L2(v);

        sm4_to_xor_2((unsigned char *)(k + i), (unsigned char *)(&w), (unsigned char *)(k + i + 4), 4);

        RK[i] = k[i + 4];
    }

    return 0;
}

int sm4_one_enc(unsigned int *mk, unsigned int *in, unsigned int *out)
{
    unsigned int x[36];
    unsigned int u, v, w;
    int i = 0;
    int j = 0;

    x[0] = in[0];
    x[1] = in[1];
    x[2] = in[2];
    x[3] = in[3];

    sm4_get_rki(mk);

    for (i = 0; i < 32; i++)
    {
        sm4_to_xor_4((unsigned char *)(x + i + 1), (unsigned char *)(x + i + 2), (unsigned char *)(x + i + 3), (unsigned char *)(RK + i), (unsigned char *)(&u), 4);

        sm4_to_tao((unsigned char *)(&u), (unsigned char *)(&v), 4);

        w = L1(v);

        sm4_to_xor_2((unsigned char *)(x + i), (unsigned char *)(&w), (unsigned char *)(x + i + 4), 4);

        x[i + 4];
    }

    out[0] = x[35];
    out[1] = x[34];
    out[2] = x[33];
    out[3] = x[32];

    return 0;
}

int sm4_one_dec(unsigned int *mk, unsigned int *in, unsigned int *out)
{
    unsigned int x[36];
    unsigned int u, v, w;
    int i = 0;
    int j = 0;

    x[0] = in[0];
    x[1] = in[1];
    x[2] = in[2];
    x[3] = in[3];

    sm4_get_rki(mk);

    for (i = 0; i < 32; i++)
    {
        sm4_to_xor_4((unsigned char *)(x + i + 1), (unsigned char *)(x + i + 2), (unsigned char *)(x + i + 3), (unsigned char *)(RK + 31 - i), (unsigned char *)(&u), 4);

        sm4_to_tao((unsigned char *)(&u), (unsigned char *)(&v), 4);

        w = L1(v);

        sm4_to_xor_2((unsigned char *)(x + i), (unsigned char *)(&w), (unsigned char *)(x + i + 4), 4);

        x[i + 4];
    }

    out[0] = x[35];
    out[1] = x[34];
    out[2] = x[33];
    out[3] = x[32];

    return 0;
}

int main()
{
    // unsigned char mk[16] = "1234567890abcdef";
    unsigned int mk[4] = {1, 1, 4,5};
    unsigned int mm[4]= {0xB60DE9B3, 0x7A7A7B4C, 0xC7D03789, 0x2A278E6C};
    unsigned char *b = (unsigned char *)mm;
    unsigned char c[16] = {0};

    sm4_one_dec((unsigned int *)mk, (unsigned int *)b, (unsigned int *)c);
    for (size_t i = 0; i < 16; i++)
    {
        printf("%c", c[i]);
    }

    return 0;
}
```
