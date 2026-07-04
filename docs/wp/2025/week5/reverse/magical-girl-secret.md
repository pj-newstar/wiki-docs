---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 魔法少女的秘密

使用 [Bluter](https://github.com/worawit/blutter) 解包 `flutter`。
用 `blutter` 解出来，然后比较明显一个加密一个调用

打开 `asm` 文件夹看 `drink.dart`

```dart
// lib: , url: package:magical_girl/drink.dart

// class id: 1048975, size: 0x8
class :: {
}

// class id: 186, size: 0x8, field offset: 0x8
//   const constructor,
class drink extends Object {

  _ encrypt(/* No info */) {
    // --asm--
  }
  _ _encryptUint32List(/* No info */) {
    // --asm--
  }
  _ _fixkey(/* No info */) {
    // --asm--
  }
  _ _toUint32List(/* No info */) {
    // --asm--
  }
}
```

全是汇编，所以直接去 IDA 中运行脚本，再搜索函数 `encrypt` 可以找到。

`magical_girl_drink_drink::encrypt_297350()` 函数

```cpp
  *(_QWORD *)(v0 - 16) = v5;
  *(_QWORD *)(v0 - 8) = v6;
  v7 = v0 - 16;
  v8 = (_QWORD *)(v0 - 72);
  if ( (unsigned __int64)v8 <= *(_QWORD *)(v2 + 56) )
    ((void (*)(void))StackOverflowSharedWithoutFPURegsStub_3b1f64)();
  v9 = *(_QWORD *)(v3 + 3792);
  *v8 = *(_QWORD *)(v7 + 24);
  v8[1] = v9;
  *(_QWORD *)(v7 - 8) = dart_convert_Utf8Encoder::convert_309850();
  v10 = *(_QWORD *)(v3 + 3792);
  *v11 = *(_QWORD *)(v3 + 46752);
  v11[1] = v10;
  v12 = dart_convert_Utf8Encoder::convert_309850();
  result = *(_QWORD *)(v7 - 8);
  *(_QWORD *)(v7 - 16) = v12;
  if ( *(_DWORD *)(result + 19) )
  {
    v15 = *(_QWORD *)(v7 + 32);
    v13[1] = result;
    v13[2] = v15;
    *v13 = v1 + 32;
    *(_QWORD *)(v7 - 8) = magical_girl_drink_drink::_toUint32List_297aec();
    v16 = *(_QWORD *)(v7 + 32);
    *v17 = *(_QWORD *)(v7 - 16);
    v17[1] = v16;
    v18 = magical_girl_drink_drink::_fixkey_297908();
    v19 = *(_QWORD *)(v7 + 32);
    v20[1] = v18;
    v20[2] = v19;
    *v20 = v1 + 48;
    v21 = magical_girl_drink_drink::_toUint32List_297aec();
    v22 = *(_QWORD *)(v7 + 32);
    v23[1] = *(_QWORD *)(v7 - 8);
    v23[2] = v22;
    *v23 = v21;
    v24 = ((__int64 (*)(void))magical_girl_drink_drink::_encryptUint32List_2974e0)();
    // ..
  }
```

从代码来看，程序会将 FLAG 和 `key` 都转换为 `u32`，再进入加密函数 `magical_girl_drink_drink::_encryptUint32List_2974e0`。

看一下上层调用 `magical_girl_EditView_MyEditTextState::check_2972a8`

```c
__int64 __fastcall magical_girl_EditView_MyEditTextState::check_2972a8(
        __int64 input,
        __int64 a2,
        __int64 a3,
        __int64 a4,
        __int64 a5,
        __int64 a6,
        __int64 a7)
{
  __int64 v7; // x15
  __int64 v8; // x26
  _QWORD *v9; // x27
  __int64 v10; // x28
  __int64 v11; // x29
  __int64 v12; // x30
  __int64 v13; // x29
  _QWORD *v14; // x15
  __int64 v15; // x16
  __int64 v16; // x0
  __int64 *v17; // x15
  __int64 v18; // x1
  __int64 v19; // x16
  __int64 *v20; // x15
  __int64 v21; // x1
  __int64 v22; // x2

  *(_QWORD *)(v7 - 16) = v11;
  *(_QWORD *)(v7 - 8) = v12;
  v13 = v7 - 16;
  v14 = (_QWORD *)(v7 - 48);
  if ( (unsigned __int64)v14 <= *(_QWORD *)(v8 + 56) )
    StackOverflowSharedWithoutFPURegsStub_3b1f64(input, a2, a3, a4, a5, a6, a7);
  v15 = v9[5843];
  v14[1] = *(_QWORD *)(v13 + 16);
  v14[2] = v15;
  *v14 = v9[5844];
  v16 = magical_girl_drink_drink::encrypt_297350();
  *(_QWORD *)(v13 - 8) = v16;
  *v17 = v16;
  dart_core_::print_1aa6a4();
  v18 = *(unsigned int *)(*(_QWORD *)(v13 + 24) + 27LL) + (v10 << 32);
  v19 = v9[260];
  v20[1] = *(_QWORD *)(v13 - 8);
  v20[2] = v19;
  *v20 = v18;
  if ( (flutter_src_foundation_collections_::listEquals_1d58b8() & 0x10) != 0 )
  {
    v21 = *(_QWORD *)(v13 + 24);
    v22 = v9[5846];
  }
  else
  {
    v21 = *(_QWORD *)(v13 + 24);
    v22 = v9[5845];
  }
  *(_DWORD *)(v21 + 19) = v22;
  return v9[62];
}
```

这里应该是与密文进行判断

```c
if ( (flutter_src_foundation_collections_::listEquals_1d58b8() & 0x10) != 0 )
```

找到这个函数 https://api.flutter.dev/flutter/foundation/listEquals.html

传入 2 个 list，猜测其中一个为加密的输入，另一个就是密文，分析完成。

现在先用 `blutter` 给出的 `frida` 脚本 `hook` 一下加密函数 `magical_girl_drink_drink::_encryptUint32List_2974e0`

```js
const ShowNullField = false;
const MaxDepth = 5;
var libapp = null;

function pr(_this, idx) {
  init(_this.context);
  let objPtr = getArg(_this.context, idx);
  const [tptr, cls, values] = getTaggedObjectValue(objPtr);
  console.log(`${cls.name}@${tptr.toString().slice(2)} =`, JSON.stringify(values, null, 2));
}

function onLibappLoaded() {
  // xxx("remove this line and correct the hook value");
  const fn_addr = 0x2974e0;
  Interceptor.attach(libapp.add(fn_addr), {
    onEnter: function (args) {
      pr(this, 0);
      pr(this, 1);
    },
    onLeave: function (retval) {
      pr(this, 1);
    },
  });
}

function tryLoadLibapp() {
  try {
    libapp = Module.findBaseAddress("libapp.so");
  } catch (e) {
    if (e instanceof TypeError && e.message === "not a function") {
      libapp = Process.findModuleByName("libapp.so");
      if (libapp != null) {
        libapp = libapp.base;
      }
    } else {
      throw e;
    }
  }
  if (libapp === null) setTimeout(tryLoadLibapp, 500);
  else onLibappLoaded();
}
tryLoadLibapp();
// more is being skipped
```

在root机中打开应用

```shell
frida -U -F work.pangbai.magic.magical_girl -l blutter_frida.js -o out.txt
```

可以看到输入的测试密文和密钥以及被加密的输入

```js
_Uint32List@6e00817cd9 = [
  1936287828,
  1264677705,
  1870100837,
  1870098295
]
_Uint32List@6e00817c89 = [
  943011893,
  4
]

// 函数返回时
_Uint32List@6e00817c89 = [
  496308832,
  3081350031
]
```

第一个数组转为字符串就是 `ThisIsaKeywowowo` 肯定是 `Key` 了

分析一下这个加密函数

```cpp
__int64 __usercall magical_girl_drink_drink::_encryptUint32List_2974e0@<X0>(
        __int64 a1@<X0>,
        __int64 a2@<X1>,
        __int64 a3@<X3>,
        __int64 a4@<X4>,
        __int64 a5@<X5>,
        __int64 a6@<X6>,
        __int64 a7@<X7>,
        __int64 a8@<X8>)
{
  __int64 v8; // x15
  __int64 v9; // x26
  __int64 v10; // x27
  __int64 v11; // x29
  __int64 v12; // x30
  __int64 v13; // x29
  _QWORD *v14; // x15
  __int64 v15; // x2
  __int64 v16; // x3
  __int64 v17; // x5
  __int64 v18; // x6
  __int64 v19; // x2
  __int64 v20; // x4
  signed __int64 MintSharedWithoutFPURegsStub_3b20e4; // x0
  __int64 v22; // x2
  signed __int64 v23; // x1
  __int64 v24; // x0
  int v25; // w2
  __int64 v26; // x2
  __int64 v27; // x1
  __int64 v28; // x2
  _QWORD *v29; // x15
  __int64 v30; // x4
  __int64 v31; // x3
  __int64 v32; // x5
  __int64 v33; // x0
  __int64 v34; // x6
  __int64 v35; // x11
  __int64 v36; // x7
  __int64 v37; // x10
  __int64 v38; // x8
  signed __int64 v39; // x0
  __int64 v40; // x3
  __int64 v41; // x4
  __int64 v42; // x1
  __int64 v43; // x0
  __int64 v44; // x0
  __int64 v45; // x2
  __int64 v46; // x1
  unsigned __int64 v47; // x0
  __int64 v48; // x3
  __int64 v49; // x1
  __int64 v50; // x2
  __int64 v51; // x16
  __int64 v52; // x0
  __int64 v54; // x0
  __int64 v55; // x5
  __int64 v56; // x6
  __int64 v57; // x15
  __int64 v58; // x2
  __int64 v59; // x3
  __int64 v60; // x0

  *(_QWORD *)(v8 - 16) = v11;
  *(_QWORD *)(v8 - 8) = v12;
  v13 = v8 - 16;
  v14 = (_QWORD *)(v8 - 128);
  v15 = 52;
  if ( (unsigned __int64)v14 <= *(_QWORD *)(v9 + 56) )
    StackOverflowSharedWithoutFPURegsStub_3b1f64(a1, a2, 52, a3, a4, a5, a6);
  v16 = *(_QWORD *)(v13 + 24);
  v17 = (__int64)*(int *)(v16 + 19) >> 1;
  *(_QWORD *)(v13 - 56) = v17;
  v18 = v17 - 1;
  *(_QWORD *)(v13 - 48) = v17 - 1;
  if ( !v17 )
  {
    v54 = ((__int64 (*)(void))RangeErrorSharedWithoutFPURegsStub_3b23ac)();
    *(_QWORD *)(v57 - 16) = v55;
    *(_QWORD *)(v57 - 8) = v56;
    v57 -= 16;
    *(_QWORD *)(v57 - 16) = v58;
    *(_QWORD *)(v57 - 8) = v59;
    *(_QWORD *)(v57 - 24) = v54;
    v24 = (*(__int64 (**)(void))(v9 + 504))();
    __break(0);
LABEL_28:
    StackOverflowSharedWithoutFPURegsStub_3b1f64(v24, v23, v26, v16, v20, v17, v18);
    goto LABEL_8;
  }
  v19 = v15 / v17 + 6;
  v20 = *(unsigned int *)(v16 + 4 * v18 + 23);
  MintSharedWithoutFPURegsStub_3b20e4 = 2 * (int)v19;
  if ( v19 != MintSharedWithoutFPURegsStub_3b20e4 >> 1 )
  {
    MintSharedWithoutFPURegsStub_3b20e4 = AllocateMintSharedWithoutFPURegsStub_3b20e4(v19, v16, v20, v17, v18, a7, a8);
    *(_QWORD *)(MintSharedWithoutFPURegsStub_3b20e4 + 7) = v22;
  }
  v23 = MintSharedWithoutFPURegsStub_3b20e4;
  v24 = *(_QWORD *)(v13 + 16);
  v25 = *(_DWORD *)(v24 + 19);
  *(_QWORD *)(v13 - 40) = (__int64)v25 >> 1;
  *(_QWORD *)(v13 - 32) = (__int64)v25 >> 1;
  v26 = 0;
  while ( 2 )
  {
    *(_QWORD *)(v13 - 8) = v20;
    *(_QWORD *)(v13 - 16) = v26;
    *(_QWORD *)(v13 - 24) = v23;
    if ( (unsigned __int64)v14 <= *(_QWORD *)(v9 + 56) )
      goto LABEL_28;
LABEL_8:
    *v14 = 0;
    v14[1] = v23;
    if ( ((*(__int64 (__fastcall **)(_QWORD))(v10 + 46784))(v14[1]) & 0x10) != 0 )
      return *(_QWORD *)(v13 + 24);
    v30 = 1131796;
    v31 = 3;
    v32 = (unsigned int)*(_QWORD *)(v13 - 16) + 1131796;
    *(_QWORD *)(v13 - 96) = v32;
    v33 = (unsigned int)v32 >> 2;
    v34 = v33 & 3;
    *(_QWORD *)(v13 - 88) = v34;
    v35 = *(_QWORD *)(v13 - 8);
    v36 = *(_QWORD *)(v13 + 24);
    v37 = 0;
    while ( 1 )
    {
      v38 = *(_QWORD *)(v13 - 48);
      *(_QWORD *)(v13 - 72) = v35;
      *(_QWORD *)(v13 - 80) = v37;
      if ( (unsigned __int64)v29 <= *(_QWORD *)(v9 + 56) )
        StackOverflowSharedWithoutFPURegsStub_3b1f64(v33, v27, v28, 3, 1131796, v32, v34);
      v39 = 2 * (int)v37;
      if ( v37 != v39 >> 1 )
      {
        v39 = AllocateMintSharedWithoutFPURegsStub_3b20e4(v28, v31, v30, v32, v34, v36, v38);
        *(_QWORD *)(v39 + 7) = v37;
      }
      *(_QWORD *)(v13 - 64) = v39;
      if ( v37 >= v38 )
        break;
      *(_QWORD *)(v13 - 16) = v37 + 1;
      *(_QWORD *)(v13 - 8) = *(unsigned int *)(v36 + 4 * (v37 + 1) + 23);
      if ( (v39 & 1) != 0 && (unsigned __int64)((unsigned int)*(_QWORD *)(v39 - 1) >> 12) - 59 > 1 )
        IsType_int_Stub_3cf048();
      v40 = *(_QWORD *)(v13 + 24);
      v41 = *(_QWORD *)(v13 - 80);
      v28 = *(unsigned int *)(v40 + 4 * v41 + 23);
      if ( (v41 & 3 ^ (unsigned __int64)(unsigned int)*(_QWORD *)(v13 - 88)) >= *(_QWORD *)(v13 - 40) )
      {
        v43 = ((__int64 (*)(void))RangeErrorSharedWithoutFPURegsStub_3b23ac)();
        goto LABEL_30;
      }
      v42 = (unsigned int)*(_QWORD *)(v13 - 8);
      LODWORD(v33) = v28
                   + ((((*(__int64 *)(v13 - 72) >> 5) ^ (4 * v42)) + ((v42 >> 3) ^ (16 * *(_QWORD *)(v13 - 72))))
                    ^ ((*(_QWORD *)(v13 - 96) ^ v42)
                     + (*(_DWORD *)(*(_QWORD *)(v13 + 16)
                                  + 4 * (*(_QWORD *)(v13 - 80) & 3LL ^ (unsigned int)*(_QWORD *)(v13 - 88))
                                  + 23)
                      ^ *(_QWORD *)(v13 - 72))));
      v27 = v40 + 4 * v41;
      *(_DWORD *)(v27 + 23) = v33;
      v33 = (unsigned int)v33;
      v35 = (unsigned int)v33;
      v37 = *(_QWORD *)(v13 - 16);
      v36 = v40;
      v32 = *(_QWORD *)(v13 - 96);
      v34 = *(_QWORD *)(v13 - 88);
      v31 = 3;
      v30 = 1131796;
    }
    v43 = *(_QWORD *)(v13 - 56);
    if ( !v43 )
    {
LABEL_30:
      v47 = RangeErrorSharedWithoutFPURegsStub_3b23ac(v43);
      break;
    }
    *(_QWORD *)(v13 - 16) = *(unsigned int *)(v36 + 23);
    v44 = *(_QWORD *)(v13 - 64);
    *(_QWORD *)(v13 - 8) = *(unsigned int *)(v36 + 4 * v38 + 23);
    if ( (v44 & 1) != 0 && (unsigned __int64)((unsigned int)*(_QWORD *)(v44 - 1) >> 12) - 59 > 1 )
      IsType_int_Stub_3cf048();
    v47 = *(_QWORD *)(v13 - 32);
    if ( (*(_QWORD *)(v13 - 80) & 3LL ^ (unsigned __int64)(unsigned int)*(_QWORD *)(v13 - 88)) < v47 )
    {
      v46 = (unsigned int)*(_QWORD *)(v13 - 16);
      v45 = *(_QWORD *)(v13 - 72);
      v48 = (unsigned int)*(_QWORD *)(v13 - 8)
          + ((((unsigned int)(v45 >> 5) ^ (4 * (_DWORD)v46)) + ((unsigned int)(v46 >> 3) ^ (16 * (_DWORD)v45)))
           ^ (((unsigned int)*(_QWORD *)(v13 - 96) ^ (unsigned int)v46)
            + (*(_DWORD *)(*(_QWORD *)(v13 + 16)
                         + 4 * (*(_QWORD *)(v13 - 80) & 3LL ^ (unsigned int)*(_QWORD *)(v13 - 88))
                         + 23)
             ^ (unsigned int)v45)));
      v49 = *(_QWORD *)(v13 + 24);
      v50 = *(_QWORD *)(v13 - 48);
      *(_QWORD *)(v13 - 16) = v48;
      *(_DWORD *)(v49 + 4 * v50 + 23) = v48;
      v51 = *(_QWORD *)(v13 - 24);
      *v29 = 2;
      v29[1] = v51;
      v52 = (*(__int64 (__fastcall **)(_QWORD, __int64, __int64, __int64, _QWORD, _QWORD))(v10 + 46832))(
              v29[1],
              v49,
              v50,
              v48,
              0,
              *(_QWORD *)(v10 + 46824));
      v26 = (unsigned int)*(_QWORD *)(v13 - 96);
      v20 = (unsigned int)*(_QWORD *)(v13 - 16);
      v23 = v52;
      v16 = *(_QWORD *)(v13 + 24);
      v24 = *(_QWORD *)(v13 + 16);
      v18 = *(_QWORD *)(v13 - 48);
      v17 = *(_QWORD *)(v13 - 56);
      continue;
    }
    break;
  }
  v60 = RangeErrorSharedWithoutFPURegsStub_3b23ac(v47);
  return magical_girl_drink_drink::_fixkey_297908(v60);
}
```

应该是一个 `XXTEA`，`1131796` 的 16 进制为 `0x114514`，还有加法操作，可以知道只更改了 `DELTA=0x114514`。

解密算法没有问题了，现在只需要找到密文。刚刚分析了对比密文和输入的位置是在函数 `flutter_src_foundation_collections_::listEquals_1d58b8()` 进行的，使用 `frida hook` 查看传入的参数。

```js
const ShowNullField = false;
const MaxDepth = 5;
var libapp = null;

function pr(_this, idx) {
  init(_this.context);
  let objPtr = getArg(_this.context, idx);
  const [tptr, cls, values] = getTaggedObjectValue(objPtr);
  console.log(`${cls.name}@${tptr.toString().slice(2)} =`, JSON.stringify(values, null, 2));
}

function onLibappLoaded() {
  // xxx("remove this line and correct the hook value");
  const fn_addr = 0x1d58b8;
  Interceptor.attach(libapp.add(fn_addr), {
    onEnter: function (args) {
      pr(this, 0);
      pr(this, 1);
    },
  });
}

function tryLoadLibapp() {
  try {
    libapp = Module.findBaseAddress("libapp.so");
  } catch (e) {
    if (e instanceof TypeError && e.message === "not a function") {
      libapp = Process.findModuleByName("libapp.so");
      if (libapp != null) {
        libapp = libapp.base;
      }
    } else {
      throw e;
    }
  }
  if (libapp === null) setTimeout(tryLoadLibapp, 500);
  else onLibappLoaded();
}
tryLoadLibapp();
// more is being skipped
```

这次 `hook` 的输出很多，但是我找到了加密后的输入，另一个参数应该就是密文。

```js
// more text
_Uint8List@6e005e5671 = [
  134,
  139,
  225,
  194,
  10,
  134,
  7,
  131,
  88,
  191,
  107,
  222,
  52,
  133,
  87,
  112,
  194,
  148,
  71,
  228,
  191,
  70,
  1,
  147,
  123,
  52,
  228,
  254,
  20,
  111,
  81,
  245,
  110,
  92,
  93,
  159,
  129,
  240,
  231,
  205
]
_Uint8List@6e006438b9 = [
  96,
  18,
  149,
  29,
  143,
  171,
  169,
  183
]
// more text
```

对这个密文进行解密操作。

```cpp
#include <stdint.h>
#define DELTA 0x114514
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
unsigned char keys[] = "ThisIsaKeywowowo";
unsigned char mm[] = {134,139,225,194,10,134,7,131,88,191,107,222,52,133,87,112,194,148,71,228,191,70,1,147,123,52,228,254,20,111,81,245,110,92,93,159,129,240,231,205};
int main()
{
    btea((uint32_t *)mm, -10, (uint32_t *)keys);
    unsigned char *p = mm;
    for (size_t i = 0; i < 40; i++)
    {
        printf("%c", p[i]);
    }

    return 0;
}

```

如果想深入思考本题请参考 [Want2BecomeMagicalGirl2025](https://pangbai.work/CTF/Reverse/Want2BecomeMagicalGirl/)
