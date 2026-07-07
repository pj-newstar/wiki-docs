---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# Dancing Keys

### 分析程序逻辑

查看程序的 main 函数：

```cpp
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  unsigned int v4; // [rsp+8h] [rbp-B8h]
  int i; // [rsp+Ch] [rbp-B4h]
  int j; // [rsp+10h] [rbp-B0h]
  int v7; // [rsp+18h] [rbp-A8h] BYREF
  int v8; // [rsp+1Ch] [rbp-A4h]
  _DWORD v9[12]; // [rsp+20h] [rbp-A0h] BYREF
  char v10[48]; // [rsp+50h] [rbp-70h] BYREF
  char s[56]; // [rsp+80h] [rbp-40h] BYREF
  unsigned __int64 v12; // [rsp+B8h] [rbp-8h]

  v12 = __readfsqword(0x28u);
  v4 = 0xFFFFFFFF;
  do
  {
    switch ( v4 )
    {
      case 0xFFFFFFFF:
        dword_404070 = 0x12345678;
        v4 = 0xFFFF8000;
        break;
      case 0xFFFF8000:
        dword_404074 = 0x90ABCDEF;
        v4 = 0xFFF98000;
        break;
      case 0x973A7229:
        dword_40407C = 0x19810114;
        v4 = 0x9FBAF32D;
        break;
      default:
        dword_404078 = 0x11451419;
        v4 = 0x973A7229;
        break;
    }
  }
  while ( v4 != 0x9FBAF32D );
  puts("Input your FLAG: ");
  __isoc99_scanf("%48s", s);
  if ( (unsigned int)strlen(s) == 48 )
  {
    sub_40132B(s, v9, 48);
    for ( i = 0; i <= 11; i += 2 )
    {
      v7 = v9[i];
      v8 = v9[i + 1];
      sub_4015EB(&v7, &dword_404070);
      v9[i] = v7;
      v9[i + 1] = v8;
    }
    sub_4013F3(v9, v10, 12);
    for ( j = 0; j <= 47; ++j )
    {
      if ( v10[j] != byte_402020[j] )
      {
        puts("Wrong FLAG!");
        return 114514;
      }
    }
    puts("Right FLAG!");
    return 1919810;
  }
  else
  {
    puts("Wrong length!");
    return 114514;
  }
}
```

我们发现，main在开头给4个dword进行了赋值，随后在 `sub_4015EB` 中使用了这组dword，作为加密的密钥，我们查看这个函数：

```cpp
__int64 __fastcall sub_4015EB(unsigned int *a1, _DWORD *a2)
{
  __int64 result; // rax
  unsigned int v3; // [rsp+18h] [rbp-18h]
  unsigned int v4; // [rsp+1Ch] [rbp-14h]
  int v5; // [rsp+20h] [rbp-10h]
  unsigned int i; // [rsp+24h] [rbp-Ch]
  int v7; // [rsp+28h] [rbp-8h]

  v3 = *a1;
  v4 = a1[1];
  v5 = 0xDEADBEEF;
  v7 = rand();
  for ( i = 0; i < 114; ++i )
  {
    v5 += v7 * i;
    v3 += (v4 - v5) ^ a2[2] ^ (v4 << ((char)(i + 1) % 5)) ^ (v4 >> ((i + 4) & 7)) ^ *a2;
    v4 += (v3 - v5) ^ a2[1] ^ (v3 << ((char)(i + 2) % 6)) ^ (v3 >> ((int)(i + 3) % 7)) ^ a2[3];
  }
  *a1 = v3;
  result = v4;
  a1[1] = v4;
  return result;
}
```

发现是一个初始sum值不为0，delta为随机值的魔改TEA加密，那么之前的4个连续dword自然就是密钥了，`sub_40132B` 和 `sub_4013F3` 则是将byte与dword互相转化的函数。

但是实际动态调试时我们发现，key似乎并不是这个初始化的值，且 `sub_40132B` 和 `sub_4013F3` 不是标准的大/小端序转换，而是进行了自定义的重排：

```cpp
__int64 __fastcall sub_40132B(__int64 a1, __int64 a2, int a3)
{
  __int64 result; // rax
  int i; // [rsp+20h] [rbp-4h]

  for ( i = 0; ; ++i )
  {
    result = (unsigned int)(a3 / 4);
    if ( i >= (int)result )
      break;
    *(_DWORD *)(4LL * i + a2) = (*(unsigned __int8 *)(4 * i + 2LL + a1) << 16)
                              | *(unsigned __int8 *)(4 * i + 3LL + a1)
                              | (*(unsigned __int8 *)(4 * i + a1) << 8)
                              | (*(unsigned __int8 *)(4 * i + 1LL + a1) << 24);
  }
  return result;
}

__int64 __fastcall sub_4013F3(__int64 a1, __int64 a2, int a3)
{
  __int64 result; // rax
  unsigned int i; // [rsp+20h] [rbp-4h]

  for ( i = 0; ; ++i )
  {
    result = i;
    if ( (int)i >= a3 )
      break;
    *(_BYTE *)((int)(4 * i) + 2LL + a2) = *(_DWORD *)(4LL * (int)i + a1);
    *(_BYTE *)((int)(4 * i) + 3LL + a2) = BYTE1(*(_DWORD *)(4LL * (int)i + a1));
    *(_BYTE *)((int)(4 * i) + a2) = BYTE2(*(_DWORD *)(4LL * (int)i + a1));
    *(_BYTE *)((int)(4 * i) + 1LL + a2) = HIBYTE(*(_DWORD *)(4LL * (int)i + a1));
  }
  return result;
}
```

除此之外，经过细心观察似乎还能发现，key的值貌似是会变化的，与函数上打入的断点密切相关，因此可以猜测这是某种反调试逻辑；另一方面，如果delta是随机值，则几乎大概率在其之前有一个 `srand(seed)` 来设置随机数种子，但我们查看所有的函数，发现并没有srand函数的引入，rand函数也不是粉色的库函数，我们查看rand的实现：

```cpp
__int64 rand()
{
  dword_404060 ^= dword_404060 << 11;
  dword_404060 ^= (unsigned int)dword_404060 >> 4;
  dword_404060 ^= 32 * dword_404060;
  dword_404060 ^= (unsigned int)dword_404060 >> 14;
  return (unsigned int)dword_404060;
}
```

原来这个rand函数是一个自实现的随机函数，`dword_404060` 则是全局的随机数种子，我们查看其交叉引用（xref），发现除了rand之外，其还在一个叫 `start_routine` 的函数中被引用，查看这个函数：

```cpp
void *__fastcall start_routine(void *a1)
{
  int i; // [rsp+1Ch] [rbp-14h]

  dword_404060 = sub_40128A(start, 4200941LL - (_QWORD)start);
  sleep(1u);
  for ( i = 0; i <= 3; ++i )
    dword_404070[i] ^= rand();
  return 0;
}
```

可以看到，`seed` 是将 `start` 函数开始的地址一直到某个偏移量（这里是程序 `.text` 段的结尾）位置的字节带入 `sub_40128A` 中生成的，且 `start_routine` 同时还修改了 `dword_404070`（也就是密钥）的值，查看 `sub_40128A`：

```cpp
__int64 __fastcall sub_40128A(__int64 a1, unsigned __int64 a2)
{
  int v3; // [rsp+10h] [rbp-20h]
  int v4; // [rsp+14h] [rbp-1Ch]
  unsigned __int64 i; // [rsp+18h] [rbp-18h]

  v3 = 0;
  v4 = 0;
  for ( i = 0; i < a2; ++i )
  {
    v3 += i * *(unsigned __int8 *)(a1 + i);
    v4 ^= *(unsigned __int8 *)(a1 + i);
  }
  return v3 * (v4 ^ (unsigned int)ptrace(PTRACE_TRACEME, 0, 0, 0));
}
```

这是一个自实现的简单校验函数，将程序的整个 `.text` 段的所有字节计算一个哈希，并加入了反调试逻辑。打入函数上的断点实际上改变了运行的程序在内存中的字节，因此如果断点的位置、数量等不同，每次生成的 `key` 值也会不同。

那么 `start_routine` 这个函数究竟是什么时候被调用的呢？动态调试可以发现其是在 `main` 之前被调用的。查看其 xref，有一个函数 `sub_401590` 调用了它：

```cpp
unsigned __int64 sub_401590()
{
  pthread_t newthread; // [rsp+0h] [rbp-10h] BYREF
  unsigned __int64 v2; // [rsp+8h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  pthread_create(&newthread, 0, start_routine, 0);
  pthread_detach(newthread);
  return v2 - __readfsqword(0x28u);
}
```

这个函数创建了一个线程，使其与 `main` 同时运行。`start_routine` 中的 `sleep(1)` 用于等待 `main` 中的 key 初始化赋值。`sub_401590` 的 xref 位于 `.init_array` 段；在 ELF 文件中，该段中的函数会先于 `main` 执行，因此这就是 key 被篡改的原因。

既然存在反调试和哈希校验，还能否拿到密钥？

可以，这里有两种方法：

方法 1：在哈希生成函数中的 `ptrace` 处下断点。由于 `ptrace` 是库函数，其调用位置在 `.plt.sec` 段，在 `.text` 段上方，不在哈希函数的处理范围内，因此可以在其上打断点：

```asm
.plt.sec:00000000004010F0
.plt.sec:00000000004010F0 ; =============== S U B R O U T I N E =======================================
.plt.sec:00000000004010F0
.plt.sec:00000000004010F0 ; Attributes: thunk
.plt.sec:00000000004010F0
.plt.sec:00000000004010F0 ; __int64 ptrace(enum __ptrace_request request, ...)
.plt.sec:00000000004010F0 _ptrace         proc near               ; CODE XREF: sub_40128A+8B↓p
.plt.sec:00000000004010F0                 endbr64
.plt.sec:00000000004010F4                 jmp     cs:off_404020 ;在这里打断点
.plt.sec:00000000004010F4 _ptrace         endp
.plt.sec:00000000004010F4
.plt.sec:00000000004010F4 ; ---------------------------------------------------------------------------
```

断在这里之后单步调试，或者直接运行到鼠标指针处（Run to cursor）：

```asm
.text:00000000004012F2
.text:00000000004012F2 loc_4012F2:                             ; CODE XREF: sub_40128A+32↑j
.text:00000000004012F2                 mov     rax, [rbp+var_18]
.text:00000000004012F6                 cmp     rax, [rbp+var_30]
.text:00000000004012FA                 jb      short loc_4012BE
.text:00000000004012FC                 mov     ecx, 0
.text:0000000000401301                 mov     edx, 0
.text:0000000000401306                 mov     esi, 0
.text:000000000040130B                 mov     edi, 0          ; request
.text:0000000000401310                 mov     eax, 0
.text:0000000000401315                 call    _ptrace ;这里是调用的地方
.text:000000000040131A                 mov     [rbp+var_8], rax ;断点之后可以运行到这里来
.text:000000000040131E                 mov     rax, [rbp+var_8]
.text:0000000000401322                 xor     eax, [rbp+var_1C]
.text:0000000000401325                 imul    eax, [rbp+var_20]
.text:0000000000401329                 leave
.text:000000000040132A                 retn
.text:000000000040132A ; } // starts at 40128A
.text:000000000040132A sub_40128A      endp
.text:000000000040132A
```

在这里观察 `rax` 寄存器，其值就是 `ptrace` 返回值，为 `0xFFFFFFFFFFFFFFFF`（-1）。将其改为 `0`，即可返回正确的哈希值。

方法 2：手动求哈希。使用十六进制编辑器或 IDA 直接 dump `.text` 段，再手动编写脚本求解哈希值。该方法较繁琐，且不容易确认结果正确性，更适合作为辅助验证手段。

### 实现加密解密

得到哈希后，就可以复现并逆向程序的加密逻辑。先对 byte 与 dword 互相转换的函数进行逆向：

```python
def b2d(input_bytes):
    output = []
    for i in range(0, len(input_bytes), 4):
        b0 = input_bytes[i + 0]
        b1 = input_bytes[i + 1]
        b2 = input_bytes[i + 2]
        b3 = input_bytes[i + 3]
        value = (b3 << 0) | (b0 << 8) | (b2 << 16) | (b1 << 24)
        output.append(value)
    return output

def re_d2b(input_bytes):
    output = []
    for i in range(0, len(input_bytes), 4):
        b0 = input_bytes[i + 0]
        b1 = input_bytes[i + 1]
        b2 = input_bytes[i + 2]
        b3 = input_bytes[i + 3]
        value = (b2 << 0) | (b3 << 8) | (b0 << 16) | (b1 << 24)
        output.append(value)
    return output

def d2b(input_uint32_list):
    output = bytearray()
    for val in input_uint32_list:
        byte0 = (val >>  0) & 0xFF
        byte1 = (val >>  8) & 0xFF
        byte2 = (val >> 16) & 0xFF
        byte3 = (val >> 24) & 0xFF
        output.append(byte2)
        output.append(byte3)
        output.append(byte0)
        output.append(byte1)
    return bytearray(output)

def re_b2d(input_uint32_list):
    output = bytearray()
    for val in input_uint32_list:
        byte3 = (val >>  0) & 0xFF
        byte0 = (val >>  8) & 0xFF
        byte2 = (val >> 16) & 0xFF
        byte1 = (val >> 24) & 0xFF
        output.append(byte0)
        output.append(byte1)
        output.append(byte2)
        output.append(byte3)
    return bytearray(output)
```

然后复现正向加密逻辑并编写逆向脚本（这里也包含手动求哈希的方法）：

```python
op = bytearray.fromhex("f30f1efa31ed4989d15e4889e24883e4f050544531c031c948c7c760174000ff15832e0000f4662e0f1f840000000000f30f1efac3662e0f1f84000000000090b850404000483d504040007413b8000000004885c07409bf50404000ffe06690c366662e0f1f8400000000000f1f4000be504040004881ee504040004889f048c1ee3f48c1f8034801c648d1fe7411b8000000004885c07407bf50404000ffe0c366662e0f1f8400000000000f1f4000f30f1efa803d652e0000007513554889e5e87affffffc605532e0000015dc390c366662e0f1f8400000000000f1f4000f30f1efaeb8af30f1efa554889e58b053c2e0000c1e00b89c28b05312e000031d08905292e00008b05232e0000c1e80489c28b05182e000031d08905102e00008b050a2e0000c1e00589c28b05ff2d000031d08905f72d00008b05f12d0000c1e80e89c28b05e62d000031d08905de2d00008b05d82d00005dc3f30f1efa554889e54883ec3048897dd8488975d0c745e000000000c745e400000000488b45d8488945f048c745e800000000eb34488b55f0488b45e84801d00fb6000fb6c0488b55e80fafc20145e0488b55f0488b45e84801d00fb6000fb6c03145e4488345e801488b45e8483b45d072c2b900000000ba00000000be00000000bf00000000b800000000e8d6fdffff488945f8488b45f83345e40faf45e0c9c3f30f1efa554889e548897de8488975e08955dcc745fc00000000e98e0000008b45fcc1e0024898488d5003488b45e84801d00fb6000fb6c08b55fcc1e2024863ca488b55e84801ca0fb6120fb6d2c1e20809c28b45fcc1e0024898488d4802488b45e84801c80fb6000fb6c0c1e01089d109c18b45fcc1e0024898488d5001488b45e84801d00fb6000fb6c0c1e01889c28b45fc4898488d348500000000488b45e04801f009ca89108345fc018b45dc8d500385c00f48c2c1f8023945fc0f8c5bffffff90905dc3f30f1efa554889e548897de8488975e08955dcc745fc00000000e9c40000008b45fc4898488d148500000000488b45e84801d08b088b45fcc1e0024898488d5002488b45e04801d089ca88108b45fc4898488d148500000000488b45e84801d08b00c1e80889c18b45fcc1e0024898488d5003488b45e04801d089ca88108b45fc4898488d148500000000488b45e84801d08b00c1e81089c18b45fcc1e0024863d0488b45e04801d089ca88108b45fc4898488d148500000000488b45e84801d08b00c1e81889c18b45fcc1e0024898488d5001488b45e04801d089ca88108345fc018b45fc3b45dc0f8c30ffffff90905dc3f30f1efa554889e54883ec3048897dd8488d0533fcffff488945f0488d05e5040000488d1521fcffff4829d0488945f8488b55f8488b45f04889d64889c7e861fdffff8905312b0000bf01000000e8e7fbffffc745ec00000000eb41b800000000e8cafcffff8b55ec4863d2488d0c9500000000488d150f2b00008b141131d089c18b45ec4898488d148500000000488d05f42a0000890c028345ec01837dec037eb9b800000000c9c3f30f1efa554889e54883ec1064488b042528000000488945f831c0488d45f0b900000000488d152bffffffbe000000004889c7e838fbffff488b45f04889c7e80cfbffff90488b45f864482b0425280000007405e8e7faffffc9c3f30f1efa554889e54883ec3048897dd8488975d0488b45d88b008945e8488b45d88b40048945ecc745f0efbeaddeb800000000e8f3fbffff8945f8c745fc72000000c745f400000000e9020100008b45f40faf45f80145f08b45f48d48014863c14869c06766666648c1e82089c2d1fa89c8c1f81f29c289d0c1e00201d029c189ca8b45ec89d1d3e089c2488b45d04883c0088b0031c28b45ec2b45f089d631c68b45f483c00483e0078b55ec89c1d3ea488b45d08b0031d031f00145e88b45f48d48024863c14869c0abaaaa2a48c1e8204889c289c8c1f81f29c289d001c001d001c029c189ca8b45e889d1d3e089c2488b45d04883c0048b0031c28b45e82b45f089d631c68b45f483c0034863d04869d29324499248c1ea2001c2c1fa0289c1c1f91f29ca89d1c1e10329d129c889c28b45e889d1d3e889c2488b45d04883c00c8b0031d031f00145ec8345f4018b45f43b45fc0f82f2feffff488b45d88b55e88910488b45d8488d50048b45ec890290c9c3f30f1efa554889e54881ecc000000064488b042528000000488945f831c0c78548ffffffffffffff83bd48ffffffff743281bd48ffffff0080ffff743981bd48ffffff0080ffff777b81bd48ffffff29723a97745a81bd48ffffff0080f9ff7438eb61c705a328000078563412c1a548ffffff0feb4ec70594280000efcdab908b9548ffffff89d001c001d0c1e00201d0898548ffffffeb2bc705752800001914451181b548ffffff29f2c368eb15c7056328000014018119818d48ffffff0ce1908d9081bd48ffffff2df3ba9f7405e953ffffff90488d05130800004889c7e86bf8ffff488d45c04889c6488d050f0800004889c7b800000000e8b0f8ffff488d45c04889c7e854f8ffff898554ffffff83bd54ffffff307419488d05e50700004889c7e826f8ffffb852bf0100e933010000488d8d60ffffff488d45c0ba300000004889ce4889c7e87cfaffffc7854cffffff00000000eb7a8b854cffffff48988b848560ffffff898558ffffff8b854cffffff83c00148988b848560ffffff89855cffffff488d8558ffffff488d157a2700004889d64889c7e8eafcffff8b9558ffffff8b854cffffff489889948560ffffff8b854cffffff83c0018b955cffffff489889948560ffffff83854cffffff0283bd4cffffff0b0f8e79ffffff488d4d90488d8560ffffffba0c0000004889ce4889c7e896faffffc78550ffffff00000000eb418b8550ffffff48980fb65405908b8550ffffff4898488d0d9b0600000fb6040838c27416488d05e10600004889c7e814f7ffffb852bf0100eb24838550ffffff0183bd50ffffff2f7eb6488d05c70600004889c7e8eef6ffffb8424b1d00488b55f864482b1425280000007405e8f5f6ffffc9c3000000f30f1efa4883ec084883c408c3")
hash_ttl = 0
xor_ttl = 0

for i in range(len(op)):
    hash_ttl += op[i] * i
    xor_ttl ^= op[i]

final_hash = (hash_ttl * xor_ttl) & 0xFFFFFFFF
print(hex(final_hash))
stored_hash = final_hash

def rand():
    global final_hash
    final_hash = (final_hash ^ (final_hash << 11)) & 0xFFFFFFFF
    final_hash = (final_hash ^ (final_hash >> 4)) & 0xFFFFFFFF
    final_hash = (final_hash ^ (final_hash << 5)) & 0xFFFFFFFF
    final_hash = (final_hash ^ (final_hash >> 14)) & 0xFFFFFFFF
    return final_hash

key = [0x12345678, 0x90ABCDEF, 0x11451419, 0x19810114]

for i in range(4):
    key[i] ^= rand()
    print(hex(key[i]),end=" ")
print()

inp = bytearray(b"123456781234567812345678123456781234567812345678")
msg = b2d(inp)
result = []
for j in range(0,len(msg)//2):
    m1 = msg[2*j]
    m2 = msg[2*j+1]
    ttl = 0xDEADBEEF
    delta = rand()
    rounds = 114
    # print(ttl)
    # print(ttl, delta, rounds)
    for i in range(rounds):
        ttl = (ttl + i * delta) & 0xFFFFFFFF
        m1 = (m1 + (((m2 << ((i + 1) % 5)) ^ key[2]) ^ (m2 - ttl) ^ ((m2 >> ((i + 4) % 8)) ^ key[0]))) & 0xFFFFFFFF
        m2 = (m2 + (((m1 << ((i + 2) % 6)) ^ key[1]) ^ (m1 - ttl) ^ ((m1 >> ((i + 3) % 7)) ^ key[3]))) & 0xFFFFFFFF
    result.append(m1)
    result.append(m2)
enc = d2b(result)
print(enc.hex())
```

确认输出与程序比较时的结果相同后即可解密。需要注意以下几点：

1. 需要重新计算一次哈希值，确保使用的种子一致，并且要手动调用 4 次 `rand` 来模拟修改密钥时的 `rand` 行为。

2. 注意 `sum` 值的起始值，加的不是 `轮数 * delta`，因为每轮加的是 `i * delta` 而不是 `delta`。

3. 注意每组解密时轮数需要倒序，直接正向解密是错误的，因为 `sum` 每次变化的值与轮数有关。

解密脚本：

```python
cipher = bytearray.fromhex("6eaab24614a47e60ba444ecc43aaaacdd4fc71aaf67d4b9be67def4e3d430bbf281485b2cf62a2c5ea7deb5ed6fc3cbf")
denc = re_d2b(cipher)
final_hash = stored_hash
# print(hex(final_hash))
for i in range(4):
    rand()
res = []

for j in range(0, len(denc)//2):
    m1 = denc[2*j]
    m2 = denc[2*j+1]
    ttl_base = 0xDEADBEEF
    delta = rand()
    rounds = 114
    # print(ttl_base, delta, rounds)
    ttl = (ttl_base + ((rounds-1) * rounds // 2) * delta) & 0xFFFFFFFF
    # print(ttl)
    for i in range(rounds - 1, -1, -1):
        m2 = (m2 - (((m1 << ((i + 2) % 6)) ^ key[1]) ^ (m1 - ttl) ^ ((m1 >> ((i + 3) % 7)) ^ key[3]))) & 0xFFFFFFFF
        m1 = (m1 - (((m2 << ((i + 1) % 5)) ^ key[2]) ^ (m2 - ttl) ^ ((m2 >> ((i + 4) % 8)) ^ key[0]))) & 0xFFFFFFFF
        ttl = (ttl - i * delta) & 0xFFFFFFFF
    # print(ttl)
    res.append(m1)
    res.append(m2)
dec = re_b2d(res)
print(dec)
# bytearray(b'FLAG{1_h4t3_h4sH_ch3cK1ng_4Nd_r4Nd0m_t3AenCRypt}')
```

FLAG：

```plaintext
flag{1_h4t3_h4sH_ch3cK1ng_4Nd_r4Nd0m_t3AenCRypt}
```
