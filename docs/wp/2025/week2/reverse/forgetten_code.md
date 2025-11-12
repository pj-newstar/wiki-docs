---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# Forgetten_Code

附件是一个 `.s` 文件。

> [!INFO]
> **.s 文件是什么？**
> 
> 是 GCC 编译器从 C/C++ 源代码生成的汇编源代码文件。它包含了低级指令<span data-desc>（x86 汇编）</span>，但还是可读的文本形式。GCC 的编译过程包括预处理、编译<span data-desc>（生成 .s）</span>、汇编<span data-desc>（生成 .o）</span>和链接<span data-desc>（生成可执行文件）</span>。

用 `gcc chal.s -o chal` 将 .s 文件汇编并链接成可执行文件 chal 后，拖入 IDA 分析。

```cpp
unsigned int *__fastcall fn(unsigned int *a1)
{
  unsigned int *result; // rax
  unsigned int j; // [rsp+1Ch] [rbp-14h]
  int v3; // [rsp+20h] [rbp-10h]
  unsigned int v4; // [rsp+24h] [rbp-Ch]
  unsigned int v5; // [rsp+28h] [rbp-8h]
  int i; // [rsp+2Ch] [rbp-4h]

  for ( i = 0; i <= 15; ++i )
    *((_BYTE *)ng + i) ^= 0x11u;
  v5 = *a1;
  v4 = a1[1];
  v3 = 0;
  for ( j = 0; j <= 31; ++j )
  {
    v3 -= 0x61C88647;
    v5 += (v4 + v3) ^ (ng[0] + 16 * v4) ^ ((v4 >> 5) + ng[1]);
    v4 += (v5 + v3) ^ (ng[2] + 16 * v5) ^ ((v5 >> 5) + ng[3]);
  }
  *a1 = v5;
  result = a1 + 1;
  a1[1] = v4;
  return result;
}
```

可以看到加密过程是一个 TEA 加密，不过每次加密的前会对密钥的每一个字节异或 `0x11`。

exp 如下：

```cpp
#include <cstring>
#include <stdio.h>

char key[] = {0x73, 0x70, 0x7f, 0x76, 0x75, 0x63, 0x74, 0x70,
              0x7c, 0x78, 0x65, 0x62, 0x7c, 0x68, 0x76, 0x7e};

unsigned int cipher[] = {0x482550ff, 0x2a60a11e, 0xfa9d5db7, 0x8b4b2a10,
                         0xd38d2d8e, 0x04092276, 0xfad7e1ba, 0x98f9c1b9};

void tea_decrypt(unsigned int* v) {
    // 和加密流程一样，先处理密钥
    for (int i = 0; i < 16; i++) { key[i] ^= 0x11; }

    // 后面进入标准的 TEA 解密
    unsigned int v0 = v[0], v1 = v[1], sum = 0x9e3779b9 * 32, i;
    unsigned int delta = 0x9e3779b9;

    for (i = 0; i < 32; i++) {
        unsigned int k0 = ((unsigned int*)key)[0], k1 = ((unsigned int*)key)[1],
                     k2 = ((unsigned int*)key)[2], k3 = ((unsigned int*)key)[3];
        v1 -= ((v0 << 4) + k2) ^ (v0 + sum) ^ ((v0 >> 5) + k3);
        v0 -= ((v1 << 4) + k0) ^ (v1 + sum) ^ ((v1 >> 5) + k1);

        sum -= delta;
    }
    v[0] = v0;
    v[1] = v1;
}

int main() {
    for (int i = 0; i < 4; i++) { tea_decrypt(cipher + 2 * i); }

    printf("flag{");
    for (int i = 0; i < 32; i++) { printf("%c", ((char*)cipher)[i]); }
    printf("}\n");
    return 0;
}

// flag{4553m81y_5_s0o0o0_345y_jD5yQ5mD9}
```

