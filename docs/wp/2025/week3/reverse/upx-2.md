---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 尤皮·埃克斯历险记（2）

UPX 标识符改了，和正常的相比改了 UPX012!和版本号，改回去就行了，可以和 UPX1 cmp 一下
改完脱壳查看主函数：

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  _main(*(_QWORD *)&argc, argv, envp);
  printf("Input your FLAG: ");
  scanf("%48s", Str);
  if ( strlen(Str) != 48 )
  {
    puts("Wrong length!");
    exit_0(0);
  }
  Size = strlen(Str);
  Block = malloc(Size);
  memset(Block, 0, Size);
  encrypt((unsigned __int8 *)Str, Size, Block);
  for ( n47 = 0; n47 <= 47; ++n47 )
  {
    if ( *((_BYTE *)Block + n47) != target[n47] )
    {
      puts("Wrong FLAG!");
      exit_0(0);
    }
  }
  puts("Right FLAG!");
  free(Block);
  return 0;
}
```

FLAG 长度为 48，查看加密函数：

```cpp
void __fastcall encrypt(unsigned __int8 *p_Str, size_t Size, _BYTE *a3)
{
  Count = (Size + 3) >> 2;
  i_1 = (Count + 2) / 3;
  Block = malloc(4 * Count);
  v12 = calloc(Count, 4u);
  for ( Count_1 = 0; Count_1 < Count; ++Count_1 )
  {
    Block[Count_1] = 0;
    if ( Size >= 4 * (Count_1 + 1) )
      n4 = 4;
    else
      n4 = Size - 4 * Count_1;
    for ( n4_1 = 0; n4_1 < n4; ++n4_1 )
      Block[Count_1] |= p_Str[4 * Count_1 + n4_1] << (8 * n4_1);
  }
  for ( i = 0; i < i_1; ++i )
  {
    memset(v8, 0, 0x12Cu);
    for ( n2 = 0; n2 <= 2; ++n2 )
    {
      Count_2 = 3 * i + n2;
      if ( Count_2 >= Count )
        v8[n2] = 0;
      else
        v8[n2] = Block[Count_2];
    }
    for ( n71 = 0; n71 <= 71; ++n71 )
    {
      v3 = v8[n71];
      v4 = v3 ^ RoundFunc((unsigned int)(v8[n71 + 2] ^ v8[n71 + 1]) ^ keys[n71]);
      v5 = n71 + 3;
      v8[v5] = RoundFunc(v4);
    }
    for ( n2_1 = 0; n2_1 <= 2; ++n2_1 )
    {
      Count_3 = 3 * i + n2_1;
      if ( Count_3 < Count )
        v12[Count_3] = v8[n2_1 + 72];
    }
  }
  for ( i_2 = 0; i_2 < i_1; ++i_2 )
  {
    for ( n2_2 = 0; n2_2 <= 2; ++n2_2 )
    {
      Count_4 = 3 * i_2 + n2_2;
      if ( Count_4 >= Count )
        break;
      v6 = &v12[Count_4];
      if ( i_2 )
        v7 = *v6 ^ v12[Count_4 - 3];
      else
        v7 = *v6 ^ IV[n2_2];
      *v6 = v7;
    }
  }
  for ( Count_5 = 0; Count_5 < Count; ++Count_5 )
  {
    *(_WORD *)&a3[4 * Count_5] = v12[Count_5];
    a3[4 * Count_5 + 2] = BYTE2(v12[Count_5]);
    a3[4 * Count_5 + 3] = HIBYTE(v12[Count_5]);
  }
  free(Block);
  free(v12);
}
```

是一个自定义加密，一次加密12字节，其中有多组key和3个iv，查看轮函数：

```cpp
uint32_t RoundFunc(uint32_t a) {
	unsigned int tag = 0x23fb19ad;
	while(1){
	    switch(tag){
	        case 0x23fb19ad:
	                a -= 0xC0295960U;
	            tag /= 7;
	            break;
	        case 0x0523df18:
	                a ^= 0xFD714A3EU;
	            tag += 0x4157359f;
	            break;
	        case 0x467b14b7:
	                a -= 0x3744080FU;
	            tag |= 0x5f5244ff;
	            break;
	        case 0x5f7b54ff:
	                a += 0xE51E16E4U;
	            tag /= 9;
	            break;
	        case 0x0a9becff:
	                a -= 0x5707CB50U;
	            tag >>= 9;
	            break;
	        case 0x00054df6:
	                a -= 0xFD4A0953U;
	            tag &= 0xbbe92a36;
	            break;
	        case 0x00010836:
	                a -= 0x89FF5F44U;
	            tag -= 0x68b172a5;
	            break;
	        case 0x974f9591:
	                a -= 0x6215BBF4U;
	            tag ^= 0x5a17b182;
	            break;
	        case 0xcd582413:
	                a += 0x2D48CA6FU;
	            tag |= 0xcb4fc7ce;
	            break;
	        case 0xcf5fe7df:
	                a += 0xC3254D0AU;
	            tag *= 7;
	            break;
	        case 0xab9f5719:
	                a -= 0xBA272CADU;
	            tag |= 0x1755de91;
	            break;
	        case 0xbfdfdf99:
	                a -= 0x1D13CB21U;
	            tag /= 7;
	            break;
	        case 0x1b691ff1:
	                a -= 0xD53FCF4CU;
	            tag |= 0x37b972c0;
	            break;
	        case 0x3ff97ff1:
	                a -= 0x3D6D0903U;
	            tag += 0x4f721c31;
	            break;
	        case 0x8f6b9c22:
	                a ^= 0x1756F5FDU;
	            tag ^= 0x7ba43bea;
	            break;
	        case 0xf4cfa7c8:
	                a -= 0x706BD3C7U;
	            tag -= 0x4c6629b6;
	            break;
	        case 0xa8697e12:
	                a -= 0x3C2028B7U;
	            tag <<= 16;
	            break;
	        case 0x7e120000:
	                a -= 0xC5BE1781U;
	            tag -= 0xab5724e9;
	            break;
	        case 0xd2badb17:
	                a &= 0xFFFFFFFFU;
	            tag &= 0x93da8979;
	            break;
	    }
	    if(tag == 0x929a8911){
	        break;
	    }
	}
    return a;
}
```

d810是去不掉这种混淆的，可以把函数复制出来，将含有a的行包裹上printf，这样打印出来的就是按顺序执行的加密：

```cpp
a -= 0xC0295960;
a ^= 0xFD714A3E;
a -= 0x3744080F;
a += 0xE51E16E4;
a -= 0x5707CB50;
a -= 0xFD4A0953;
a -= 0x89FF5F44;
a -= 0x6215BBF4;
a += 0x2D48CA6F;
a += 0xC3254D0A;
a -= 0xBA272CAD;
a -= 0x1D13CB21;
a -= 0xD53FCF4C;
a -= 0x3D6D0903;
a ^= 0x1756F5FD;
a -= 0x706BD3C7;
a -= 0x3C2028B7;
a -= 0xC5BE1781;
```

编写脚本同构加密和解密即可：

```py
def b2nle(b, n):
    return [int.from_bytes(b[i:i+n], byteorder='little', signed=False) for i in range(0, len(b), n)]

def n2ble(na, n):
    b = bytearray()
    for a in na: b.extend(a.to_bytes(n, byteorder='little'))
    return b

keys = [0xd344fc67, 0x2210bdb7, 0x76bb9c00, 0x53f1b5de, 0x821a977f, 0xf5b01673, 0x2a406627, 0x935f493c, 0xb98347c1, 0xe1ad274a, 0xf68b39ce, 0xbcb77109,
        0xae8207af, 0x54f52f5a, 0x2487acb7, 0x2baa52bd, 0xd7a45b9f, 0xb93d82c7, 0x77fbf041, 0x1747530c, 0x7ea63dee, 0x8bad0343, 0x38822bd3, 0x806b9e9d,
        0x242525cf, 0x1f5d96be, 0x1adb4554, 0x47b628d0, 0x77c9a358, 0x3c43d913, 0x711165d3, 0x1afdea6e, 0x57ef6f26, 0x75cdb37e, 0xf08680de, 0x7eaad562,
        0x7aba9243, 0x45af3320, 0xf7f816b2, 0x3dd5c8d1, 0x6d8251f6, 0x7606e5d0, 0x38dced31, 0x7fa1260b, 0xbaeff202, 0xd9d85e1d, 0x5e583700, 0x35dfcc5f,
        0x1b689abb, 0x1b2bbb67, 0xcf506375, 0x3a3d4268, 0x46a5141b, 0x7fe3136c, 0x3e86f672, 0x8a0b8eee, 0x33d87cd7, 0xd4a50ea9, 0xc77afcdd, 0xcdc0d74d,
        0xe0b6f0bc, 0x66c0e9c7, 0xd494b811, 0x9d1d8a81, 0x147c00b6, 0xdf60c3e4, 0x5fa112f8, 0x7186229a, 0x7fdcdc37, 0x1435fe6b, 0xf97112a5, 0xea79306c]

IV = [0xbe87e8b2, 0x88e9f392, 0x16fb40c3]

def RoundFunc(a):
    a -= 0xC0295960
    a ^= 0xFD714A3E
    a -= 0x3744080F
    a += 0xE51E16E4
    a -= 0x5707CB50
    a -= 0xFD4A0953
    a -= 0x89FF5F44
    a -= 0x6215BBF4
    a += 0x2D48CA6F
    a += 0xC3254D0A
    a -= 0xBA272CAD
    a -= 0x1D13CB21
    a -= 0xD53FCF4C
    a -= 0x3D6D0903
    a ^= 0x1756F5FD
    a -= 0x706BD3C7
    a -= 0x3C2028B7
    a -= 0xC5BE1781
    a &= 0xFFFFFFFF
    return a

def inv_RF(a):
    a += 0xC5BE1781
    a += 0x3C2028B7
    a += 0x706BD3C7
    a ^= 0x1756F5FD
    a += 0x3D6D0903
    a += 0xD53FCF4C
    a += 0x1D13CB21
    a += 0xBA272CAD
    a -= 0xC3254D0A
    a -= 0x2D48CA6F
    a += 0x6215BBF4
    a += 0x89FF5F44
    a += 0xFD4A0953
    a += 0x5707CB50
    a -= 0xE51E16E4
    a += 0x3744080F
    a ^= 0xFD714A3E
    a += 0xC0295960
    a &= 0xFFFFFFFF
    return a

binp = bytearray(b"")
dinp = b2nle(binp, 4)
dres = [0] * len(dinp)
for i in range(0, len(dinp), 3):
    mes = dinp[i:i+3] + [0]*72
    for j in range(72): mes[j+3] = RoundFunc(mes[j] ^ RoundFunc(mes[j+1] ^ mes[j+2] ^ keys[j]))
    dres[i:i+3] = mes[72:75]
for i in range(0, len(dinp) // 3):
    if i == 0:
        for j in range(3): dres[i*3+j] ^= IV[j]
    else:
        for j in range(3): dres[i*3+j] ^= dres[i*3+j-3]
bres = n2ble(dres, 4)
print(bres.hex())

bres = bytearray.fromhex("c7c94c956fbfc9f4c486a42057556be2eadcb73f9c421ee172820d93b3f9d0359370ff44726155f8ecdafb6ea8a6cb9e")
dres = b2nle(bres, 4)
dinp = [0] * len(dres)
for i in range(len(dres) // 3 - 1, -1, -1):
    if i == 0:
        for j in range(3): dres[i*3+j] ^= IV[j]
    else:
        for j in range(3): dres[i*3+j] ^= dres[i*3+j-3]
for i in range(len(dres)-3, -3, -3):
    mes = [0]*72 + dres[i:i+3]
    for j in range(71, -1, -1): mes[j] = inv_RF(mes[j+3]) ^ RoundFunc(mes[j+1] ^ mes[j+2] ^ keys[j])
    dinp[i:i+3] = mes[0:3]
binp = n2ble(dinp, 4)
print(binp.decode())
```

`flag{CoN7r0l_F10w_F14t73n1nG_C4N_b3_c0NfU5iNg!!}`
