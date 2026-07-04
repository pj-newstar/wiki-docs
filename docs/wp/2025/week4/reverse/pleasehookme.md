---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# PleaseHookMe

## 分析

打开 `so` 搜索 `Java` ，因为编译器优化的关系，看起来代码有点恐怖。
`veorq_s8` 和 `vqtbl1q_s8` 都是 `ARM64` 的 `Neon` 指令，用来生成密钥，可以看看 `ARM` 的手册，写循环来模拟这个算法。本题更推荐使用 `Frida` `HOOK`

```cpp
bool __fastcall Java_work_pangbai_hookme_FirstFragment_check(__int64 *a1, __int64 a2, __int64 a3)
{
  __int64 v3; // x8
  const char *s; // x20
  size_t v5; // x0
  char *dest; // x19
  char *s_1; // x0
  _BOOL4 v8; // w20
  int v9; // w13
  unsigned int v10; // w14
  unsigned int mm; // w8
  unsigned int v12; // w10
  int8x16_t v13; // q2
  unsigned int v14; // w9
  unsigned int v15; // w11
  int v16; // w15
  bool v17; // cf
  int v18; // w16
  int8x16_t v20; // [xsp+0h] [xbp-20h] BYREF
  _BYTE v21[4]; // [xsp+14h] [xbp-Ch] BYREF
  __int64 v22; // [xsp+18h] [xbp-8h]

  v22 = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  v3 = *a1;
  v21[0] = 0;
  s = (const char *)(*(__int64 (__fastcall **)(__int64 *, __int64, _BYTE *))(v3 + 1352))(a1, a3, v21);
  v5 = strlen(s);
  dest = (char *)malloc(v5 + 1);
  s_1 = strcpy(dest, s);
  v8 = 0;
  if ( strlen(s_1) == 16 )
  {
    v9 = -214;
    v10 = -1640531527;
    mm = *(_DWORD *)dest;
    v12 = *((_DWORD *)dest + 1);
    v13 = veorq_s8((int8x16_t)l, (int8x16_t)t);
    v15 = *((_DWORD *)dest + 2);
    v14 = *((_DWORD *)dest + 3);
    v20 = veorq_s8(
            vqtbl1q_s8(
              veorq_s8(
                vqtbl1q_s8(
                  veorq_s8(
                    vqtbl1q_s8(
                      veorq_s8(
                        vqtbl1q_s8(
                          veorq_s8(
                            vqtbl1q_s8(
                              veorq_s8(
                                vqtbl1q_s8(
                                  veorq_s8(
                                    vqtbl1q_s8(
                                      veorq_s8(
                                        vqtbl1q_s8(
                                          veorq_s8(
                                            vqtbl1q_s8(
                                              veorq_s8(
                                                vqtbl1q_s8(
                                                  veorq_s8(
                                                    vqtbl1q_s8(
                                                      veorq_s8(
                                                        vqtbl1q_s8(
                                                          veorq_s8(
                                                            vqtbl1q_s8((int8x16_t)xmmword_720, (int8x16_t)t),
                                                            v13),
                                                          (int8x16_t)t),
                                                        v13),
                                                      (int8x16_t)t),
                                                    v13),
                                                  (int8x16_t)t),
                                                v13),
                                              (int8x16_t)t),
                                            v13),
                                          (int8x16_t)t),
                                        v13),
                                      (int8x16_t)t),
                                    v13),
                                  (int8x16_t)t),
                                v13),
                              (int8x16_t)t),
                            v13),
                          (int8x16_t)t),
                        v13),
                      (int8x16_t)t),
                    v13),
                  (int8x16_t)t),
                v13),
              (int8x16_t)t),
            v13);
    do
    {
      v16 = (v10 >> 2) & 3;
      v17 = __CFADD__(v9++, 1);
      mm += (((4 * v12) ^ (v14 >> 5)) + ((16 * v12) ^ (v14 >> 3))) ^ ((*(_DWORD *)((unsigned __int64)&v20 & 0xFFFFFFFFFFFFFFF3LL | (4 * ((v10 >> 2) & 3LL))) ^ v14)
                                                                    + (v12 ^ v10));
      v12 += (((4 * v15) ^ (mm >> 5)) + ((16 * v15) ^ (mm >> 3))) ^ ((*(_DWORD *)((unsigned __int64)&v20 & 0xFFFFFFFFFFFFFFF3LL | (4LL * (((unsigned __int8)v16 ^ 1) & 3))) ^ mm)
                                                                   + (v15 ^ v10));
      v15 += (((4 * v14) ^ (v12 >> 5)) + ((16 * v14) ^ (v12 >> 3))) ^ ((*(_DWORD *)((unsigned __int64)&v20 & 0xFFFFFFFFFFFFFFF3LL | (4 * ((v16 ^ 2u) & 3LL))) ^ v12)
                                                                     + (v14 ^ v10));
      v18 = (*(_DWORD *)((unsigned __int64)&v20 & 0xFFFFFFFFFFFFFFF3LL | (4LL * (((unsigned __int8)v16 ^ 3) & 3))) ^ v15)
          + (mm ^ v10);
      v10 -= 1640531527;
      v14 += (((4 * mm) ^ (v15 >> 5)) + ((16 * mm) ^ (v15 >> 3))) ^ v18;
    }
    while ( !v17 );
    *(_DWORD *)dest = mm;
    *((_DWORD *)dest + 1) = v12;
    *((_DWORD *)dest + 2) = v15;
    *((_DWORD *)dest + 3) = v14;
    v8 = mm == mm && unk_316C == v12 && dword_3170 == v15 && unk_3174 == v14;
    free(dest);
  }
  return v8;
}
```

看起来是 `TEA` 系列算法（实际为 `XXTEA`）。`(*(_DWORD *)((unsigned __int64)&v20 & 0xFFFFFFFFFFFFFFF3LL | (4 * ((v10 >> 2) & 3LL)))` 这一处解引用操作是在取 `key`，因此可以使用 Frida 提取该值。

查看汇编可知，向量操作的最后一行会将 `key` 写入内存：

```cpp
.text:0000000000000D00 20 00 00 4E                 TBL             V0.16B, {V1.16B}, V0.16B ; Vector Table Lookup
.text:0000000000000D04 00 1C 22 6E                 EOR             V0.16B, V0.16B, V2.16B ; Rd = Op1 ^ Op2
.text:0000000000000D08 E0 03 80 3D                 STR             Q0, [SP,#0x40+var_40] ; Store to Memory
```

使用 Frida hook，`0xd08` 是指令偏移。

```js
var mInterval = setInterval(function () {
  var mod = Process.findModuleByName("libhookme.so");
  if (mod == null) {
    console.log("无");
    return;
  }
  clearInterval(mInterval);

  var addr = mod.base.add(0xd08);
  console.log("hook addr: " + addr);
  console.log(Instruction.parse(addr).toString());
  Interceptor.attach(addr, {
    onEnter: function (args) {
      console.log("onEnter");
      console.log(this.context.q0);
    },
  });
  //endline
}, 1);
```

运行 `frida -l .\hook.js -F -U` 附加 App，得到 `key`。

```plaintext
hook addr: 0x75447e4d08
str q0, [sp]
[23049RAD8C::HookMe ]-> onEnter
           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
00000000  4d 65 6f 77 6d 65 30 77 6d 65 6f 77 6d 65 30 77  Meowme0wmeowme0w
```

得到密钥后，即可解开后续的 `XXTEA` 加密。

```cpp
#include <stdint.h>

#define DELTA 0x9e3779b9
#define MX (((z>>5^y<<2) + (z>>3^y<<4)) ^ ((sum^y) + (key[(p&3)^e] ^ z)))
// 容易魔改
void btea(uint32_t * v, int n, uint32_t const key[4]) {
//v 为数据，n 为数据长度 (负时为解密)，key 为密钥
    uint32_t y,  z, sum;  unsigned p, rounds, e;
    if (n > 1)
        /* Coding Part */
    {
        rounds = 6 + 52 * n;
        sum = 0;
        z = v[n - 1];
        do {
            sum += DELTA;
            e = (sum >> 2) & 3;
            for (p = 0; p < n - 1; p++) {
                y = v[p + 1];
                z = v[p] += MX;
            }
            y = v[0];
            z = v[n - 1] += MX;
        } while (-- rounds );
    } else if (n < -1)
        /* Decoding Part */
    {
        n = -n;
        rounds = 6 + 52 * n;
        sum = rounds * DELTA;
        y = v[0];
        do {
            e = (sum >> 2) & 3;
            for (p = n - 1; p > 0; p--) {
                z = v[p - 1];
                y = v[p] -= MX;
            }
            z = v[n - 1];
            y = v[0] -= MX;
            sum -= DELTA;
        } while (-- rounds );
    }
}
unsigned char keys[]="Meowme0wmeowme0w";
unsigned  int mm[]={0x583f7d05,0xc4e83e36,0x481c5aaa,0xa12f85e6};
int main()
{
    btea((uint32_t*)mm,-4,(uint32_t*)keys);
    unsigned char* p =mm;
    for (size_t i = 0; i < 16; i++)
    {
       printf("%c",p[i]);
    }

    return 0;
}

```
