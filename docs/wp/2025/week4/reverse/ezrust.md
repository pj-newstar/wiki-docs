---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# ezrust

主要逻辑在 `sub_7FF718641DD0` 里面：

```cpp
__int64 sub_7FF718641DD0()
{
  // [COLLAPSED LOCAL DECLARATIONS. PRESS NUMPAD "+" TO EXPAND]

  v77 = -2;
  si128_1.m128i_i64[0] = (__int64)&off_7FF71865B6A0;// "Enter your FLAG: "
  si128_1.m128i_i64[1] = 1;
  v70.m256i_i64[0] = 8;
  *(_OWORD *)&v70.m256i_u64[1] = 0;
  sub_7FF7186467D0(&si128_1);
  expand_32_byte_kString_Theocracy[0].m128i_i64[0] = sub_7FF718645DA0();
  v0 = sub_7FF718645DE0(expand_32_byte_kString_Theocracy);
  if ( v0 )
  {
    si128_1.m128i_i64[0] = v0;
    core::result::unwrap_failed(
      (unsigned int)aCalledResultUn,
      43,
      (unsigned int)&si128_1,
      (unsigned int)&off_7FF71865B520,
      (__int64)&off_7FF71865B6C0);              // "src\\main.rs"
  }
  v66 = 0;
  v67 = 1;
  v68 = 0;
  expand_32_byte_kString_Theocracy[0].m128i_i64[0] = sub_7FF718645BA0();
  if ( (sub_7FF718645BE0(expand_32_byte_kString_Theocracy, &v66) & 1) != 0 )
  {
    si128_1.m128i_i64[0] = v1;
    core::result::unwrap_failed(
      (unsigned int)aCalledResultUn,
      43,
      (unsigned int)&si128_1,
      (unsigned int)&off_7FF71865B520,
      (__int64)&off_7FF71865B6D8);              // "src\\main.rs"
  }
  v2 = sub_7FF718642690(v67, v68);
  if ( n40 != 40 )
  {
    si128_1.m128i_i64[0] = (__int64)&off_7FF71865B730;// "Wrong FLAG! Try again!\n"
    si128_1.m128i_i64[1] = 1;
    v70.m256i_i64[0] = 8;
    *(_OWORD *)&v70.m256i_u64[1] = 0;
    result = sub_7FF7186467D0(&si128_1);
    v23 = v66;
    if ( !v66 )
      return result;
    return sub_7FF718642920(v67, v23, 1);
  }
  v4 = v2;
  nullsub_1();
  Buf1_1 = (__m128i *)j___rdl_alloc(40, 1);
  if ( !Buf1_1 )
    sub_7FF71865A453(1, 40, &off_7FF71865B5D8); // "C:\\Users\\azusa\\.rustup\\toolchains\\stable-x86_64-pc-windows-msvc\\lib/rustlib/src/rust\\library\\alloc\\src\\slice.rs"
  Buf1_1[2].m128i_i64[0] = *(_QWORD *)(v4 + 32);
  v6 = *(__m128i *)v4;
  Buf1_1[1] = _mm_loadu_si128((const __m128i *)(v4 + 16));
  Buf1 = Buf1_1;
  *Buf1_1 = v6;
  qmemcpy(
    expand_32_byte_kString_Theocracy,
    "expand 32-byte kString_Theocracy",
    sizeof(expand_32_byte_kString_Theocracy));
  v65 = _mm_loadu_si128((const __m128i *)&xmmword_7FF71865B600);
  v7 = off_7FF718665008;
  if ( *off_7FF718665008 == -1 )
    sub_7FF718659890();
  si128 = _mm_load_si128(expand_32_byte_kString_Theocracy);
  v9 = _mm_load_si128(&expand_32_byte_kString_Theocracy[1]);
  *(__m128i *)&v70.m256i_u64[2] = _mm_load_si128(&v65);
  *(__m128i *)v70.m256i_i8 = v9;
  si128_1 = si128;
  v71.m128i_i32[0] = 0;
  qmemcpy((char *)v71.m128i_i64 + 4, "NewStar 2025", 12);
  memset(v72, 0, sizeof(v72));
  v73 = 0;
  v74 = 0;
  expand_32_byte_kString_Theocracy[0].m128i_i64[0] = (__int64)Buf1;
  expand_32_byte_kString_Theocracy[0].m128i_i64[1] = (__int64)Buf1;
  expand_32_byte_kString_Theocracy[1].m128i_i64[0] = 0;
  sub_7FF718641000(&si128_1, expand_32_byte_kString_Theocracy);
  if ( *v7 == 1 )
  {
    sub_7FF718641B30(&si128_1, v72);
    v10 = _mm_load_si128(v72);
    v11 = v72[0].m128i_i8[4];
    v12 = v72[0].m128i_i8[5];
    v13 = v72[0].m128i_i8[6];
    v14 = v72[0].m128i_i8[7];
    v15 = v72[0].m128i_i8[8];
    v16 = v72[0].m128i_i8[9];
    v17 = v72[0].m128i_u8[10];
    v18 = _mm_cvtsi32_si128(*(unsigned __int16 *)((char *)&v72[0].m128i_u16[5] + 1));
    v19 = _mm_cvtsi32_si128(*(unsigned __int16 *)((char *)&v72[0].m128i_u16[6] + 1));
    v20 = _mm_cvtsi32_si128(*(unsigned __int16 *)((char *)&v72[0].m128i_u16[7] + 1));
    Buf1_2 = (char *)Buf1;
  }
  else
  {
    v24 = _mm_load_si128(&si128_1);
    v25 = _mm_load_si128((const __m128i *)&v70);
    v26 = _mm_load_si128((const __m128i *)&v70.m256i_u64[2]);
    v27 = _mm_load_si128(&v71);
    n10 = 10;
    v29 = v24;
    v30 = v25;
    v31 = v27;
    v32 = v26;
    Buf1_2 = (char *)Buf1;
    do
    {
      v33 = _mm_add_epi32(v29, v30);
      v34 = _mm_xor_si128(v31, v33);
      v35 = _mm_or_si128(_mm_slli_epi32(v34, 0x10u), _mm_srli_epi32(v34, 0x10u));
      v36 = _mm_add_epi32(v32, v35);
      v37 = _mm_xor_si128(v30, v36);
      v38 = _mm_or_si128(_mm_slli_epi32(v37, 0xCu), _mm_srli_epi32(v37, 0x14u));
      v39 = _mm_add_epi32(v33, v38);
      v40 = _mm_xor_si128(v35, v39);
      v41 = _mm_or_si128(_mm_slli_epi32(v40, 8u), _mm_srli_epi32(v40, 0x18u));
      v42 = _mm_add_epi32(v36, v41);
      v43 = _mm_xor_si128(v38, v42);
      v44 = _mm_or_si128(_mm_slli_epi32(v43, 7u), _mm_srli_epi32(v43, 0x19u));
      v45 = _mm_add_epi32(_mm_shuffle_epi32(v39, 147), v44);
      v46 = _mm_xor_si128(_mm_shuffle_epi32(v41, 78), v45);
      v47 = _mm_or_si128(_mm_slli_epi32(v46, 0x10u), _mm_srli_epi32(v46, 0x10u));
      v48 = _mm_add_epi32(_mm_shuffle_epi32(v42, 57), v47);
      v49 = _mm_xor_si128(v44, v48);
      v50 = _mm_or_si128(_mm_slli_epi32(v49, 0xCu), _mm_srli_epi32(v49, 0x14u));
      v51 = _mm_add_epi32(v45, v50);
      v52 = _mm_xor_si128(v47, v51);
      v53 = _mm_or_si128(_mm_slli_epi32(v52, 8u), _mm_srli_epi32(v52, 0x18u));
      v54 = _mm_add_epi32(v48, v53);
      v55 = _mm_xor_si128(v50, v54);
      v30 = _mm_or_si128(_mm_slli_epi32(v55, 7u), _mm_srli_epi32(v55, 0x19u));
      v32 = _mm_shuffle_epi32(v54, 147);
      v31 = _mm_shuffle_epi32(v53, 78);
      v29 = _mm_shuffle_epi32(v51, 57);
      --n10;
    }
    while ( n10 );
    v10 = _mm_add_epi32(v29, v24);
    v72[0] = v10;
    v72[1] = _mm_add_epi32(v30, v25);
    v72[2] = _mm_add_epi32(v32, v26);
    v73 = _mm_add_epi32(v31, v27);
    v71.m128i_i32[0] = _mm_cvtsi128_si32(v27) + 1;
    v75 = v10;
    v11 = v10.m128i_i8[4];
    v12 = v10.m128i_i8[5];
    v13 = v10.m128i_i8[6];
    v14 = v10.m128i_i8[7];
    v15 = v10.m128i_i8[8];
    v16 = v10.m128i_i8[9];
    v17 = v10.m128i_u8[10];
    v56 = _mm_srli_si128(v10, 14);
    v57 = _mm_shuffle_epi32(v10, 255);
    v58 = _mm_shufflelo_epi16(_mm_unpacklo_epi8(_mm_unpacklo_epi8(v56, v72[1]), (__m128i)0LL), 230);
    v20 = _mm_packus_epi16(v58, v58);
    v59 = _mm_shufflelo_epi16(_mm_unpacklo_epi8(_mm_unpacklo_epi8(v57, v56), (__m128i)0LL), 230);
    v19 = _mm_packus_epi16(v59, v59);
    v60 = _mm_shufflelo_epi16(_mm_unpacklo_epi8(_mm_unpacklo_epi8(_mm_srli_si128(v10, 10), v57), (__m128i)0LL), 230);
    v18 = _mm_packus_epi16(v60, v60);
  }
  *(_DWORD *)Buf1_2 = _mm_cvtsi128_si32(_mm_xor_si128(_mm_cvtsi32_si128(*(_DWORD *)Buf1_2), v10));
  Buf1_2[4] ^= v11;
  Buf1_2[5] ^= v12;
  Buf1_2[6] ^= v13;
  Buf1_2[7] ^= v14;
  Buf1_2[8] ^= v15;
  Buf1_2[9] ^= v16;
  v61 = _mm_load_si128((const __m128i *)&xmmword_7FF71865B4A0);
  v62 = _mm_or_si128(
          _mm_andnot_si128(v61, _mm_slli_epi64(v19, 0x18u)),
          _mm_and_si128(
            _mm_or_si128(
              _mm_slli_epi32(v18, 8u),
              _mm_andnot_si128(_mm_load_si128((const __m128i *)&xmmword_7FF71865B490), _mm_cvtsi32_si128(v17))),
            v61));
  v63 = _mm_load_si128((const __m128i *)&xmmword_7FF71865B4B0);
  *(__m128i *)(Buf1_2 + 10) = _mm_xor_si128(
                                _mm_unpacklo_epi64(
                                  _mm_or_si128(
                                    _mm_slli_epi64(_mm_cvtsi32_si128(v72[1].m128i_u8[1]), 0x38u),
                                    _mm_and_si128(
                                      _mm_or_si128(
                                        _mm_andnot_si128(v63, _mm_slli_epi64(v20, 0x28u)),
                                        _mm_and_si128(v62, v63)),
                                      (__m128i)xmmword_7FF71865B4C0)),
                                  _mm_loadl_epi64((const __m128i *)&v72[1].m128i_i16[1])),
                                _mm_loadu_si128((const __m128i *)(Buf1_2 + 10)));
  *(_QWORD *)(Buf1_2 + 26) ^= *(unsigned __int64 *)((char *)&v72[1].m128i_u64[1] + 2);
  *(_DWORD *)(Buf1_2 + 34) ^= *(unsigned __int32 *)((char *)v72[2].m128i_u32 + 2);
  *((_WORD *)Buf1_2 + 19) ^= v72[2].m128i_u16[3];
  if ( !memcmp(Buf1_2, asc_7FF71865B6F0, 0x28u) )
    si128_1.m128i_i64[0] = (__int64)&off_7FF71865B770;// "Congratulations! You found the correct FLAG!\n"
  else
    si128_1.m128i_i64[0] = (__int64)&off_7FF71865B730;// "Wrong FLAG! Try again!\n"
  si128_1.m128i_i64[1] = 1;
  v70.m256i_i64[0] = 8;
  *(_OWORD *)&v70.m256i_u64[1] = 0;
  sub_7FF7186467D0(&si128_1);
  result = sub_7FF718642920(Buf1, 40, 1);
  v23 = v66;
  if ( v66 )
    return sub_7FF718642920(v67, v23, 1);
  return result;
}
```

观察到加密过程中含有 `expand 32-byte k` 这个字符串，可以先确定为是 ChaCha20 或者 Salsa20 加密，进一步分析后可以确定是 ChaCha20 加密。

针对 ChaCha20 这种流加密，可以使用动态调试：将输入 patch 为密文并重新执行加密流程，最后得到的结果即为 FLAG 明文。该方法成立的原因是流加密的加解密过程都使用同一段密钥流进行异或。

在 `memcmp` 处找到密文为 `A36258DB4B824FCE48DABE421CD8596BC7B2CA020B216B104D4E7BEBCE9FFB21E9CF6BC2C24CB34D`。

使用 IDA 动态调试，输入一串与密文长度相同的字符，例如 `1234567890......`。在内存窗口中找到输入位置，在地址开头右键选择 `Paste Data`，填入十六进制密文后点击 `Apply`。

> 这一步中你可能需要的插件：[P4nda0s/LazyIDA: Make your IDA Lazy!](https://github.com/P4nda0s/LazyIDA)

![IDA 粘贴密文数据](/assets/images/wp/2025/week4/ezrust_1.png)

![应用密文 Patch](/assets/images/wp/2025/week4/ezrust_2.png)

之后运行到 `call memcmp` 处，查看原输入位置，即可得到 FLAG。

![调试得到 FLAG 明文](/assets/images/wp/2025/week4/ezrust_3.png)
