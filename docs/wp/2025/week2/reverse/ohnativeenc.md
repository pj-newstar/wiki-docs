---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# OhNativeEnc

<Container type='info'>

本题考查对 Android 原生加密函数的逆向分析。
</Container>

和 Week1 的安卓一样寻找加密。

![寻找加密逻辑](/assets/images/wp/2025/week2/ohnativeenc_1.png)

但是加密函数是 `native` 的，说明加密函数在 `so` 文件里。

![加密函数寻找](/assets/images/wp/2025/week2/ohnativeenc_2.png)

`.apk` 后缀改成 `.zip` 打开，把 `lib\arm64-v8a\libohnativeenc.so` 解压出来，用 `IDA` 打开。

找到 ` Java_work_pangbai_ohnativeenc_FirstFragment_checkFlag` 函数。

```cpp
bool __fastcall Java_work_pangbai_ohnativeenc_FirstFragment_checkFlag(__int64 a1, __int64 a2, __int64 a3)
{
  const char *src; // x19
  unsigned int v4; // w8
  unsigned int v5; // w10
  unsigned int v6; // w9
  unsigned int v7; // w11
  unsigned int dest__1; // w12
  unsigned int v9; // w13
  unsigned int v10; // w14
  unsigned int v11; // w15
  int v12; // w16
  unsigned int n0x1BF52; // w17
  int v14; // w1
  int v15; // w2
  int v16; // w4
  bool v17; // cf
  int v18; // w5
  int v19; // w1
  int v20; // w1
  unsigned __int64 n31; // x10
  unsigned __int64 n0x1E; // x9
  int v23; // w11
  int v24; // w12
  __int128 dest_; // [xsp+0h] [xbp-30h] BYREF
  __int128 v27; // [xsp+10h] [xbp-20h]
  __int64 v28; // [xsp+28h] [xbp-8h]

  v28 = *(_QWORD *)(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  src = (const char *)(*(__int64 (__fastcall **)(__int64, __int64, _QWORD))(*(_QWORD *)a1 + 1352LL))(a1, a3, 0LL);
  __android_log_print(4, "native", "input:%s", src);
  dest_ = 0u;
  v27 = 0u;
  strncpy((char *)&dest_, src, 0x20uLL);
  v4 = v27;
  v5 = DWORD1(v27);
  v7 = DWORD2(v27);
  v6 = HIDWORD(v27);
  dest__1 = dest_;
  v9 = DWORD1(dest_);
  v11 = DWORD2(dest_);
  v10 = HIDWORD(dest_);
  v12 = -12;
  n0x1BF52 = 0x1BF52;
  do
  {
    v14 = (n0x1BF52 >> 2) & 3;
    v15 = *(_DWORD *)&aThisisaxxteake[4 * v14];
    v16 = *(_DWORD *)&aThisisaxxteake[4 * (v14 ^ 1)];
    v17 = __CFADD__(v12++, 1);
    dest__1 += (((4 * v9) ^ (v6 >> 5)) + ((v9 >> 3) ^ (16 * v6))) ^ ((v15 ^ v6) + (v9 ^ n0x1BF52));
    v18 = *(_DWORD *)&aThisisaxxteake[4 * (v14 ^ 2)];
    v9 += (((4 * v11) ^ (dest__1 >> 5)) + ((v11 >> 3) ^ (16 * dest__1))) ^ ((v16 ^ dest__1) + (v11 ^ n0x1BF52));
    v19 = *(_DWORD *)&aThisisaxxteake[4 * (v14 ^ 3)];
    v11 += (((4 * v10) ^ (v9 >> 5)) + ((v10 >> 3) ^ (16 * v9))) ^ ((v18 ^ v9) + (v10 ^ n0x1BF52));
    v10 += (((4 * v4) ^ (v11 >> 5)) + ((v4 >> 3) ^ (16 * v11))) ^ ((v19 ^ v11) + (v4 ^ n0x1BF52));
    v4 += (((4 * v5) ^ (v10 >> 5)) + ((v5 >> 3) ^ (16 * v10))) ^ ((v15 ^ v10) + (v5 ^ n0x1BF52));
    v5 += (((4 * v7) ^ (v4 >> 5)) + ((v7 >> 3) ^ (16 * v4))) ^ ((v16 ^ v4) + (v7 ^ n0x1BF52));
    v7 += (((4 * v6) ^ (v5 >> 5)) + ((v6 >> 3) ^ (16 * v5))) ^ ((v18 ^ v5) + (v6 ^ n0x1BF52));
    v20 = (((4 * dest__1) ^ (v7 >> 5)) + ((dest__1 >> 3) ^ (16 * v7))) ^ ((v19 ^ v7) + (dest__1 ^ n0x1BF52));
    n0x1BF52 += 114514;
    v6 += v20;
  }
  while ( !v17 );
  *(_QWORD *)&dest_ = __PAIR64__(v9, dest__1);
  *((_QWORD *)&dest_ + 1) = __PAIR64__(v10, v11);
  *(_QWORD *)&v27 = __PAIR64__(v5, v4);
  *((_QWORD *)&v27 + 1) = __PAIR64__(v6, v7);
  if ( (unsigned __int8)mm[0] != (unsigned __int8)dest__1 )
    return 0LL;
  n31 = 0LL;
  do
  {
    n0x1E = n31;
    if ( n31 == 31 )
      break;
    v23 = *((unsigned __int8 *)&dest_ + n31 + 1);
    v24 = (unsigned __int8)mm[++n31];
  }
  while ( v23 == v24 );
  return n0x1E > 0x1E;
}
```

可以分析一下操作，`mm` 是密文，其实就是一个 **flag 块**加上 另一个 **flag 的加密块** 的循环，看 `key` 的内容也可以知道这是 xxtea 加密。

可以网上找个 xxtea 算法，把 `delta` 改为题目相同的值就行了。

```c
#include <stdint.h>
#define DELTA 114514
#define MX (((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4)) ^ ((sum ^ y) + (key[(p & 3) ^ e] ^ z)))
// 容易魔改
void btea(uint32_t *v, int n, uint32_t const key[4])
{
    // v 为数据，n 为数据长度 (负时为解密)，key 为密钥
    uint32_t y, z, sum;
    unsigned p, rounds, e;
    if (n > 1)
    /* Coding Part */
    {
        rounds = 6 + 52 / n;
        sum = 0;
        z = v[n - 1];
        do
        {
            sum += DELTA;
            e = (sum >> 2) & 3;
            for (p = 0; p < n - 1; p++)
            {
                y = v[p + 1];
                z = v[p] += MX;
            }
            y = v[0];
            z = v[n - 1] += MX;
        } while (--rounds);
    }
    else if (n < -1)
    /* Decoding Part */
    {
        n = -n;
        rounds = 6 + 52 / n;
        sum = rounds * DELTA;
        y = v[0];
        do
        {
            e = (sum >> 2) & 3;
            for (p = n - 1; p > 0; p--)
            {
                z = v[p - 1];
                y = v[p] -= MX;
            }
            z = v[n - 1];
            y = v[0] -= MX;
            sum -= DELTA;
        } while (--rounds);
    }
}

int main(int argc, char **argv)
{

    unsigned char key[16] = "ThisIsAXXteaKey";
    unsigned char mm[] = {0xb6, 0x53, 0x6e, 0x4d, 0x77, 0x5d, 0x08, 0xd2, 0xfb, 0x2c, 0x63, 0x1e, 0xbb, 0x7b, 0x01, 0x9b, 0xf5, 0x04, 0x6a, 0xf4, 0x0e, 0x84, 0x27, 0x47, 0x64, 0xa1, 0xe4, 0xd9, 0xef, 0x12, 0x44, 0x37};
    btea((uint32_t *)mm, -8, (uint32_t *)key);
    puts(mm);
    return 0;
}
```
