---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 尤皮·埃克斯历险记（3）

本题目抹掉了更多的 UPX 特征，可以选择带壳调试/与 week1 的最初版本进行 cmp/scylla 动态调试手动脱壳，这里以带壳调试为例

start 指向 UPX 主函数：

```cpp
// positive sp value has been detected, the output may be wrong!
void __fastcall sub_14000DFD0(__int64 a1, __int64 a2)
{
  // [COLLAPSED LOCAL DECLARATIONS. PRESS NUMPAD "+" TO EXPAND]

  v6 = v48;
  while ( 1 )
  {
    while ( 1 )
    {
      LOBYTE(a2) = *v5;
      v7 = __CFADD__((_DWORD)v2, (_DWORD)v2);
      LODWORD(v2) = 2 * v2;
      if ( !(_DWORD)v2 )
      {
        v8 = *(_DWORD *)v5;
        v7 = (unsigned __int64)v5 < 0xFFFFFFFFFFFFFFFCuLL;
        v5 += 4;
        v9 = v7 + v8;
        v7 = __CFADD__(v7, v8) | __CFADD__(v8, v9);
        LODWORD(v2) = v8 + v9;
        LOBYTE(a2) = *v5;
      }
      if ( !v7 )
        break;
      ++v5;
      *v4++ = a2;
    }
    do
    {
      v10 = v6(a1, a2);
      n3 = v10 + v7 + v10;
      v12 = __CFADD__((_DWORD)v2, (_DWORD)v2);
      v2 = (unsigned int)(2 * v2);
      if ( !(_DWORD)v2 )
      {
        v13 = *(_DWORD *)v5;
        v7 = (unsigned __int64)v5 < 0xFFFFFFFFFFFFFFFCuLL;
        v5 += 4;
        v14 = v7 + v13;
        v12 = __CFADD__(v7, v13) | __CFADD__(v13, v14);
        v2 = (unsigned int)(v13 + v14);
        LOBYTE(a2) = *v5;
      }
    }
    while ( !v12 );
    v7 = n3 < 3;
    v15 = n3 - 3;
    if ( !v7 )
      break;
LABEL_12:
    v6(a1, a2);
    v19 = v18(v17 + (unsigned int)v7 + v17);
    LODWORD(v21) = v21 + v7 + (_DWORD)v21;
    if ( !(_DWORD)v21 )
    {
      v21 = v19;
      do
      {
        v19 = v20(v21);
        v21 = v22 + (unsigned int)v7 + v22;
        v23 = __CFADD__((_DWORD)v2, (_DWORD)v2);
        LODWORD(v2) = 2 * v2;
        if ( !(_DWORD)v2 )
        {
          v24 = *(_DWORD *)v5;
          v7 = (unsigned __int64)v5 < 0xFFFFFFFFFFFFFFFCuLL;
          v5 += 4;
          v25 = v7 + v24;
          v23 = __CFADD__(v7, v24) | __CFADD__(v24, v25);
          LODWORD(v2) = v24 + v25;
        }
      }
      while ( !v23 );
    }
    ((void (__fastcall *)(_QWORD))sub_14000DF92)(v19 + (v3 < 0xFFFFFFFFFFFFF300uLL) + (_DWORD)v21);
  }
  a2 = (unsigned __int8)a2;
  ++v5;
  v16 = ~((unsigned __int8)a2 | (v15 << 8));
  if ( v16 )
  {
    v3 = v16;
    goto LABEL_12;
  }
  v48 = (__int64 (__fastcall *)(__int64, __int64))v2;
  v26 = lpflOldProtect_[0] + 11261LL;
  v27 = (_BYTE *)lpflOldProtect_[0];
  v28 = lpflOldProtect_[0];
LABEL_28:
  if ( (unsigned __int64)v27 >= v26 )
    goto LABEL_30;
  n0x8F = *v27;
  v30 = (unsigned int *)(v27 + 1);
  while ( n0x8F == 0xE8 || n0x8F == 0xE9 )
  {
    do
    {
      if ( (unsigned __int64)v30 >= v26 )
      {
LABEL_30:
        v32 = lpflOldProtect_[0];
        v33 = (char *)(lpflOldProtect_[0] + 45056LL);
        while ( 1 )
        {
          v34 = *(unsigned int *)v33;
          if ( !*(_DWORD *)v33 )
            break;
          v35 = (FARPROC *)(v32 + *((unsigned int *)v33 + 1));
          v33 += 8;
          hModule = LoadLibraryA((LPCSTR)(v34 + v32 + 57344));
          while ( 1 )
          {
            v37 = *v33++;
            if ( !v37 )
              break;
            v38 = v33;
            v39 = v33;
            v40 = v37 - 1;
            do
            {
              if ( !v38 )
                break;
              v41 = *v33++ == (char)v40;
              --v38;
            }
            while ( !v41 );
            ProcAddress = GetProcAddress(hModule, v39);
            if ( !ProcAddress )
              ExitProcess(uExitCode);
            *v35++ = ProcAddress;
          }
        }
        v44 = v33 + 4;
        for ( i = (unsigned __int64 *)(v32 - 4); ; *i = v32 + _byteswap_uint64(*i) )
        {
          LODWORD(n0xEF) = *(unsigned __int8 *)v44;
          v44 = (_WORD *)((char *)v44 + 1);
          n0xEF = (unsigned int)n0xEF;
          if ( !(_DWORD)n0xEF )
            break;
          if ( (unsigned __int8)n0xEF > 0xEFu )
          {
            LOBYTE(n0xEF) = n0xEF & 0xF;
            n0xEF = (unsigned int)((_DWORD)n0xEF << 16);
            LOWORD(n0xEF) = *v44++;
          }
          i = (unsigned __int64 *)((char *)i + n0xEF);
        }
        lpflOldProtect_[0] = 0;
        VirtualProtect((LPVOID)(v32 - 4096), 0x1000u, 4u, (PDWORD)lpflOldProtect_);
        *(_BYTE *)(v32 - 3665) &= ~0x80u;
        *(_BYTE *)(v32 - 3625) &= ~0x80u;
        VirtualProtect((LPVOID)(v32 - 4096), 0x1000u, lpflOldProtect_[0], (PDWORD)lpflOldProtect_);
        TlsCallback_0 = -4;
        ((void (__fastcall *)(__int64, __int64, _QWORD))TlsCallback_0)(v32 - 4096, 1, 0);
        do
          v50 = 0;
        while ( &v50 != &v51 - 16 );
        JUMPOUT(0x1400013F0LL);
      }
      v47 = v30;
      v31 = *v30;
      v27 = v30 + 1;
      LOBYTE(v31) = v31 - 5;
      if ( !(_BYTE)v31 )
      {
        *v47 = v28 + _byteswap_ulong(v31) - (_DWORD)v47;
        goto LABEL_28;
      }
LABEL_21:
      n0x8F = *(_BYTE *)v47;
      v30 = (unsigned int *)((char *)v47 + 1);
    }
    while ( *(_BYTE *)v47 >= 0x80u && n0x8F <= 0x8Fu && *((_BYTE *)v47 - 1) == 15 );
  }
  if ( (unsigned __int64)v30 >= v26 )
    goto LABEL_30;
  v47 = v30;
  goto LABEL_21;
}
```

注意到这里有一大段赋予内存执行权限，之后是一个大跳（JUMPOUT)，在这里就跳到了解压后的程序源代码，动态调试在这里下断点来到原程序的start：

```cpp
__int64 start_0()
{
  *(_DWORD *)qword_7FF63D315480 = 0;
  return sub_7FF63D311180();
}

__int64 sub_7FF63D311180()
{
  // [COLLAPSED LOCAL DECLARATIONS. PRESS NUMPAD "+" TO EXPAND]

  v0 = (volatile signed __int64 *)off_7FF63D3154E0;
  v1 = (void (__fastcall *)(__int64))qword_7FF63D3181B0[560];
  StackBase = NtCurrentTeb()->NtTib.StackBase;
  while ( 1 )
  {
    StackBase_1 = _InterlockedCompareExchange64(v0, (signed __int64)StackBase, 0);
    if ( !StackBase_1 )
    {
      v4 = off_7FF63D3154F0;
      v5 = 0;
      if ( *(_DWORD *)off_7FF63D3154F0 == 1 )
        goto LABEL_20;
      goto LABEL_6;
    }
    if ( StackBase == (PVOID)StackBase_1 )
      break;
    v1(1000);
  }
  v4 = off_7FF63D3154F0;
  v5 = 1;
  if ( *(_DWORD *)off_7FF63D3154F0 == 1 )
  {
LABEL_20:
    sub_7FF63D3138C0(31);
    if ( *v4 == 1 )
      goto LABEL_21;
LABEL_9:
    if ( v5 )
      goto LABEL_10;
    goto LABEL_22;
  }
LABEL_6:
  if ( *v4 )
  {
    dword_7FF63D318008 = 1;
  }
  else
  {
    v19 = off_7FF63D315530;
    *v4 = 1;
    sub_7FF63D313AA0(off_7FF63D315520, v19);
  }
  if ( *v4 != 1 )
    goto LABEL_9;
LABEL_21:
  sub_7FF63D313AA0(off_7FF63D315500, off_7FF63D315510);
  *v4 = 2;
  if ( v5 )
    goto LABEL_10;
LABEL_22:
  _InterlockedExchange64(v0, 0);
LABEL_10:
  if ( *off_7FF63D315430 )
    ((void (__fastcall *)(_QWORD, __int64, _QWORD))*off_7FF63D315430)(0, 2, 0);
  sub_7FF63D312A70();
  *(_QWORD *)qword_7FF63D3154D0 = ((__int64 (__fastcall *)(__int64 (__fastcall *)()))qword_7FF63D3181B0[559])(off_7FF63D315570);
  sub_7FF63D313AB0(nullsub_1);
  sub_7FF63D312880();
  v6 = dword_7FF63D318028;
  v7 = 8LL * (dword_7FF63D318028 + 1);
  v8 = sub_7FF63D313B18(v7);
  v9 = qword_7FF63D318020;
  v10 = v8;
  if ( v6 <= 0 )
  {
    v16 = (_QWORD *)v8;
  }
  else
  {
    v11 = v7 - 8;
    v12 = 0;
    do
    {
      v13 = sub_7FF63D3139F8(*(_QWORD *)(v9 + v12)) + 1;
      v14 = sub_7FF63D313B18(v13);
      *(_QWORD *)(v10 + v12) = v14;
      v15 = *(_QWORD *)(v9 + v12);
      v12 += 8;
      sub_7FF63D313AE0(v14, v15, v13);
    }
    while ( v11 != v12 );
    v16 = (_QWORD *)(v10 + v11);
  }
  *v16 = 0;
  qword_7FF63D318020 = v10;
  sub_7FF63D312680();
  v17 = (unsigned int)dword_7FF63D318028;
  *(_QWORD *)*off_7FF63D315440 = qword_7FF63D318018;
  result = sub_7FF63D3123D6(v17, qword_7FF63D318020);
  dword_7FF63D318010 = result;
  if ( dword_7FF63D31800C )
  {
    if ( !dword_7FF63D318008 )
    {
      sub_7FF63D313A68();
      return (unsigned int)dword_7FF63D318010;
    }
  }
  else
  {
    sub_7FF63D313280((unsigned int)result);
    *(_DWORD *)qword_7FF63D315480 = 1;
    return sub_7FF63D311180();
  }
  return result;
}
```

这里有result = xxx的就是源程序的main函数：

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  FILE *Stream; // rax
  FILE *Stream_1; // rax
  _BYTE buf[376]; // [rsp+20h] [rbp-60h] BYREF
  int v7; // [rsp+198h] [rbp+118h]
  char Str[256]; // [rsp+1A0h] [rbp+120h] BYREF
  char Buffer[256]; // [rsp+2A0h] [rbp+220h] BYREF
  unsigned __int64 n16; // [rsp+3A0h] [rbp+320h]
  size_t v11; // [rsp+3A8h] [rbp+328h]

  main_0(*(__int64 *)&argc, (__int64)argv, (__int64)envp);
  memset(buf, 0, sizeof(buf));
  v7 = 0;
  printf("Input your FLAG: ");
  Stream = (FILE *)loc_7FF63D319458(0);
  fgets(Buffer, 256, Stream);
  Buffer[strcspn(Buffer, asc_7FF63D315052)] = 0;// "\n"
  printf("Input your key: ");
  Stream_1 = (FILE *)loc_7FF63D319458(0);
  fgets(Str, 256, Stream_1);
  Str[strcspn(Str, asc_7FF63D315052)] = 0;      // "\n"
  sub_7FF63D31154A((__int64)Str);
  v11 = strlen_0(Buffer);
  sub_7FF63D311648((__int64)Buffer, (__int64)Str, buf, v11);
  n16 = (v11 + 2) / 3;
  if ( n16 == 16 && !memcmp_0(buf, &Buf2_, 0x40u) )
    printf_0("Right FLAG!");
  else
    printf_0("Wrong FLAG!");
  return 0;
}
```

`sub_7FF63D31154A` 是一个看起来像RC4 KSA初始化的函数：

```cpp
size_t __fastcall sub_7FF63D31154A(const char *Buffer)
{
  n255_2 = strlen_0(Buffer);
  n255_4 = n255_2;
  for ( n255 = 0; n255 <= 255; ++n255 )
  {
    n255_2 = n255;
    byte_7FF63D318040[n255] = n255;
  }
  v5 = 0;
  for ( n255_1 = 0; n255_1 <= 255; ++n255_1 )
  {
    v5 = (byte_7FF63D318040[n255_1] + v5 + Buffer[n255_1 % n255_4]) % 256;
    n255_3 = byte_7FF63D318040[n255_1];
    byte_7FF63D318040[n255_1] = byte_7FF63D318040[v5];
    n255_2 = n255_3;
    byte_7FF63D318040[v5] = n255_3;
  }
  return n255_2;
}
```

而其后的“加密函数”`sub_7FF63D311648` 看起来十分诡异：

```cpp
__int64 __fastcall sub_7FF63D311648(__int64 p_Buffer, __int64 Buffer, _BYTE *buf, unsigned __int64 a4)
{
  for ( i = 0; i < a4 >> 2; ++i )
    buf[i] = *(_BYTE *)(4 * i + 2 + p_Buffer);
  v136 = 0;
  v134 = 0;
  n872520205 = 872520205;
  for ( n114513 = 0; n114513 <= 114513; ++n114513 )
  {
    v133 = 1262838809 * n114513 + n872520205;
    v137 = ((v134 - v133) ^ ((v134 << 7) - 616333090) ^ ((v134 >> 2) + 1262432922)) + v136;
    v135 = ((v137 + v133) ^ (32 * v137 - 1303620281) ^ ((v137 >> 3) + 230117086)) + v134;
    n872520205 = v133 - 1262838809 * (n114513 - 1);
    v134 = v135 - ((v137 - n872520205) ^ (4 * v137 - 1162599749) ^ ((v137 >> 4) + 1952316274));
    v136 = v137 - ((v134 + n872520205) ^ ((v134 << 6) - 1429982167) ^ ((v134 >> 1) - 1212789554));
  }
  for ( n191980 = 0; n191980 <= 191980; ++n191980 )
  {
    n872520205 += 1262838809 * n191980;
    v136 += (v134 - n872520205) ^ (v134 << ((char)(n191980 + 1) % 5)) ^ (v134 >> ((n191980 + 4) & 7)) ^ 0xC84B6525;
    v134 += (v136 - n872520205) ^ (v136 << ((char)(n191980 + 2) % 6)) ^ (v136 >> ((n191980 + 3) % 7)) ^ 0x91E894EA;
  }
  buf_1 = buf;
  buf_2 = buf;
  for ( j = 0; j < a4; ++j )
  {
    *buf_1 = BYTE2(v136);
    buf_1[1] = HIBYTE(v134);
    buf_1[2] = v136;
    v4 = buf_1 + 3;
    buf_1 += 4;
    *v4 = BYTE1(v134);
  }
  for ( k = 0; k < 4 * a4; ++k )
  {
    v5 = sub_7FF63D311450((unsigned int)k);
    v6 = sub_7FF63D311450((unsigned int)(k + 1)) ^ v5;
    v7 = sub_7FF63D311450((unsigned int)(k + 2)) ^ v6;
    v8 = sub_7FF63D311450((unsigned int)(k + 3)) ^ v7;
    v9 = sub_7FF63D311450((unsigned int)(k + 4)) ^ v8;
    v10 = sub_7FF63D311450((unsigned int)(k + 5)) ^ v9;
    v11 = sub_7FF63D311450((unsigned int)(k + 6)) ^ v10;
    v12 = sub_7FF63D311450((unsigned int)(k + 7)) ^ v11;
    v13 = sub_7FF63D311450((unsigned int)(k + 8)) ^ v12;
    v14 = sub_7FF63D311450((unsigned int)(k + 9)) ^ v13;
    v15 = sub_7FF63D311450((unsigned int)(k + 10)) ^ v14;
    v16 = sub_7FF63D311450((unsigned int)(k + 11)) ^ v15;
    v17 = sub_7FF63D311450((unsigned int)(k + 12)) ^ v16;
    v18 = sub_7FF63D311450((unsigned int)(k + 13)) ^ v17;
    v19 = sub_7FF63D311450((unsigned int)(k + 14)) ^ v18;
    v20 = sub_7FF63D311450((unsigned int)(k + 15)) ^ v19;
    v21 = sub_7FF63D311450((unsigned int)(k + 16)) ^ v20;
    v22 = sub_7FF63D311450((unsigned int)(k + 17)) ^ v21;
    v23 = sub_7FF63D311450((unsigned int)(k + 18)) ^ v22;
    v24 = sub_7FF63D311450((unsigned int)(k + 19)) ^ v23;
    v25 = sub_7FF63D311450((unsigned int)(k + 20)) ^ v24;
    v26 = sub_7FF63D311450((unsigned int)(k + 21)) ^ v25;
    v27 = sub_7FF63D311450((unsigned int)(k + 22)) ^ v26;
    v28 = sub_7FF63D311450((unsigned int)(k + 23)) ^ v27;
    v29 = sub_7FF63D311450((unsigned int)(k + 24)) ^ v28;
    v30 = sub_7FF63D311450((unsigned int)(k + 25)) ^ v29;
    v31 = sub_7FF63D311450((unsigned int)(k + 26)) ^ v30;
    v32 = sub_7FF63D311450((unsigned int)(k + 27)) ^ v31;
    v33 = sub_7FF63D311450((unsigned int)(k + 28)) ^ v32;
    v34 = sub_7FF63D311450((unsigned int)(k + 29)) ^ v33;
    v35 = sub_7FF63D311450((unsigned int)(k + 30)) ^ v34;
    v36 = sub_7FF63D311450((unsigned int)(k + 31)) ^ v35;
    v37 = sub_7FF63D311450((unsigned int)(k + 32)) ^ v36;
    v38 = sub_7FF63D311450((unsigned int)(k + 33)) ^ v37;
    v39 = sub_7FF63D311450((unsigned int)(k + 34)) ^ v38;
    v40 = sub_7FF63D311450((unsigned int)(k + 35)) ^ v39;
    v41 = sub_7FF63D311450((unsigned int)(k + 36)) ^ v40;
    v42 = sub_7FF63D311450((unsigned int)(k + 37)) ^ v41;
    v43 = sub_7FF63D311450((unsigned int)(k + 38)) ^ v42;
    v44 = sub_7FF63D311450((unsigned int)(k + 39)) ^ v43;
    v45 = sub_7FF63D311450((unsigned int)(k + 40)) ^ v44;
    v46 = sub_7FF63D311450((unsigned int)(k + 41)) ^ v45;
    v47 = sub_7FF63D311450((unsigned int)(k + 42)) ^ v46;
    v48 = sub_7FF63D311450((unsigned int)(k + 43)) ^ v47;
    v49 = sub_7FF63D311450((unsigned int)(k + 44)) ^ v48;
    v50 = sub_7FF63D311450((unsigned int)(k + 45)) ^ v49;
    v51 = sub_7FF63D311450((unsigned int)(k + 46)) ^ v50;
    v52 = sub_7FF63D311450((unsigned int)(k + 47)) ^ v51;
    v53 = sub_7FF63D311450((unsigned int)(k + 48)) ^ v52;
    v54 = sub_7FF63D311450((unsigned int)(k + 49)) ^ v53;
    v55 = sub_7FF63D311450((unsigned int)(k + 50)) ^ v54;
    v56 = sub_7FF63D311450((unsigned int)(k + 51)) ^ v55;
    v57 = sub_7FF63D311450((unsigned int)(k + 52)) ^ v56;
    v58 = sub_7FF63D311450((unsigned int)(k + 53)) ^ v57;
    v59 = sub_7FF63D311450((unsigned int)(k + 54)) ^ v58;
    v60 = sub_7FF63D311450((unsigned int)(k + 55)) ^ v59;
    v61 = sub_7FF63D311450((unsigned int)(k + 56)) ^ v60;
    v62 = sub_7FF63D311450((unsigned int)(k + 57)) ^ v61;
    v63 = sub_7FF63D311450((unsigned int)(k + 58)) ^ v62;
    v64 = sub_7FF63D311450((unsigned int)(k + 59)) ^ v63;
    v65 = sub_7FF63D311450((unsigned int)(k + 60)) ^ v64;
    v66 = sub_7FF63D311450((unsigned int)(k + 61)) ^ v65;
    v67 = sub_7FF63D311450((unsigned int)(k + 62)) ^ v66;
    v68 = sub_7FF63D311450((unsigned int)(k + 63)) ^ v67;
    v69 = sub_7FF63D311450((unsigned int)(k + 64)) ^ v68;
    v70 = sub_7FF63D311450((unsigned int)(k + 65)) ^ v69;
    v71 = sub_7FF63D311450((unsigned int)(k + 66)) ^ v70;
    v72 = sub_7FF63D311450((unsigned int)(k + 67)) ^ v71;
    v73 = sub_7FF63D311450((unsigned int)(k + 68)) ^ v72;
    v74 = sub_7FF63D311450((unsigned int)(k + 69)) ^ v73;
    v75 = sub_7FF63D311450((unsigned int)(k + 70)) ^ v74;
    v76 = sub_7FF63D311450((unsigned int)(k + 71)) ^ v75;
    v77 = sub_7FF63D311450((unsigned int)(k + 72)) ^ v76;
    v78 = sub_7FF63D311450((unsigned int)(k + 73)) ^ v77;
    v79 = sub_7FF63D311450((unsigned int)(k + 74)) ^ v78;
    v80 = sub_7FF63D311450((unsigned int)(k + 75)) ^ v79;
    v81 = sub_7FF63D311450((unsigned int)(k + 76)) ^ v80;
    v82 = sub_7FF63D311450((unsigned int)(k + 77)) ^ v81;
    v83 = sub_7FF63D311450((unsigned int)(k + 78)) ^ v82;
    v84 = sub_7FF63D311450((unsigned int)(k + 79)) ^ v83;
    v85 = sub_7FF63D311450((unsigned int)(k + 80)) ^ v84;
    v86 = sub_7FF63D311450((unsigned int)(k + 81)) ^ v85;
    v87 = sub_7FF63D311450((unsigned int)(k + 82)) ^ v86;
    v88 = sub_7FF63D311450((unsigned int)(k + 83)) ^ v87;
    v89 = sub_7FF63D311450((unsigned int)(k + 84)) ^ v88;
    v90 = sub_7FF63D311450((unsigned int)(k + 85)) ^ v89;
    v91 = sub_7FF63D311450((unsigned int)(k + 86)) ^ v90;
    v92 = sub_7FF63D311450((unsigned int)(k + 87)) ^ v91;
    v93 = sub_7FF63D311450((unsigned int)(k + 88)) ^ v92;
    v94 = sub_7FF63D311450((unsigned int)(k + 89)) ^ v93;
    v95 = sub_7FF63D311450((unsigned int)(k + 90)) ^ v94;
    v96 = sub_7FF63D311450((unsigned int)(k + 91)) ^ v95;
    v97 = sub_7FF63D311450((unsigned int)(k + 92)) ^ v96;
    v98 = sub_7FF63D311450((unsigned int)(k + 93)) ^ v97;
    v99 = sub_7FF63D311450((unsigned int)(k + 94)) ^ v98;
    v100 = sub_7FF63D311450((unsigned int)(k + 95)) ^ v99;
    v101 = sub_7FF63D311450((unsigned int)(k + 96)) ^ v100;
    v102 = sub_7FF63D311450((unsigned int)(k + 97)) ^ v101;
    v103 = sub_7FF63D311450((unsigned int)(k + 98)) ^ v102;
    v104 = sub_7FF63D311450((unsigned int)(k + 99)) ^ v103;
    v105 = sub_7FF63D311450((unsigned int)(k + 100)) ^ v104;
    v106 = sub_7FF63D311450((unsigned int)(k + 101)) ^ v105;
    v107 = sub_7FF63D311450((unsigned int)(k + 102)) ^ v106;
    v108 = sub_7FF63D311450((unsigned int)(k + 103)) ^ v107;
    v109 = sub_7FF63D311450((unsigned int)(k + 104)) ^ v108;
    v110 = sub_7FF63D311450((unsigned int)(k + 105)) ^ v109;
    v111 = sub_7FF63D311450((unsigned int)(k + 106)) ^ v110;
    v112 = sub_7FF63D311450((unsigned int)(k + 107)) ^ v111;
    v113 = sub_7FF63D311450((unsigned int)(k + 108)) ^ v112;
    v114 = sub_7FF63D311450((unsigned int)(k + 109)) ^ v113;
    v115 = sub_7FF63D311450((unsigned int)(k + 110)) ^ v114;
    v116 = sub_7FF63D311450((unsigned int)(k + 111)) ^ v115;
    v117 = sub_7FF63D311450((unsigned int)(k + 112)) ^ v116;
    v118 = sub_7FF63D311450((unsigned int)(k + 113)) ^ v117;
    v119 = v118 ^ sub_7FF63D311450((unsigned int)(k + 114));
    v123 = (1 - main(argc, argv, envp)) * v119;
    v124 = buf_2++;
    *v124 ^= v123;
  }
  return 0;
}
```

不仅十分复杂，且变量设置和运算都不合理，甚至最后还出现了main函数，动态调试发现运行到 `sub_7FF63D311648` 中时会触发除零异常，检查汇编：

```asm
seg035:00007FF63D311648 push    rbp
seg035:00007FF63D311649 push    rbx
seg035:00007FF63D31164A sub     rsp, 78h
seg035:00007FF63D31164E lea     rbp, [rsp+70h]
seg035:00007FF63D311653 mov     [rbp+10h+arg_0], rcx
seg035:00007FF63D311657 mov     [rbp+10h+arg_8], rdx
seg035:00007FF63D31165B mov     [rbp+10h+arg_10], r8
seg035:00007FF63D31165F mov     [rbp+10h+arg_18], r9
seg035:00007FF63D311663 xor     eax, eax
seg035:00007FF63D311665 div     eax
seg035:00007FF63D311667 mov     rax, [rbp+10h+arg_18]
seg035:00007FF63D31166B shr     rax, 2
seg035:00007FF63D31166F mov     [rbp+10h+var_58], rax
seg035:00007FF63D311673 mov     [rbp+10h+var_18], 0
seg035:00007FF63D31167B jmp     short loc_7FF63D3116A5
```

发现确实存在除零异常，那么这个函数应该是一个垃圾函数，程序中应当有异常处理逻辑来处理这个异常，查看初始化构造函数main_0：

```cpp
__int64 __fastcall main_0(__int64 argc, __int64 argv, __int64 envp)
{
  __int64 result; // rax

  result = (unsigned int)dword_7FF63D318160;
  if ( !dword_7FF63D318160 )
  {
    dword_7FF63D318160 = 1;
    return sub_7FF63D312600(argc, argv, envp);
  }
  return result;
}

__int64 __fastcall sub_7FF63D312600(__int64 argc, __int64 argv, __int64 envp)
{
  void *v3; // rdx
  unsigned int v4; // ecx
  __int64 v5; // rax
  __int64 v6; // rcx
  void (__fastcall **v7)(__int64, void *, __int64); // rbx
  void (__fastcall **v8)(__int64, void *, __int64); // rsi
  unsigned int envp_1; // eax

  v3 = off_7FF63D3153F0;
  v4 = *(_QWORD *)off_7FF63D3153F0;
  if ( v4 == -1 )
  {
    envp_1 = 0;
    do
    {
      v4 = envp_1++;
      envp = envp_1;
    }
    while ( *((_QWORD *)off_7FF63D3153F0 + envp_1) );
  }
  if ( v4 )
  {
    v5 = v4;
    v6 = v4 - 1;
    v7 = (void (__fastcall **)(__int64, void *, __int64))((char *)off_7FF63D3153F0 + 8 * v5);
    v8 = (void (__fastcall **)(__int64, void *, __int64))((char *)off_7FF63D3153F0 + 8 * (v5 - v6) - 8);
    do
      (*v7--)(v6, v3, envp);
    while ( v7 != v8 );
  }
  return sub_7FF63D311410((__int64)sub_7FF63D3125C0, (__int64)v3, envp);
}
```

off_7FF63D3153F0指向构造函数表qword_7FF63D3155F0：

```asm
seg035:00007FF63D3155F0 qword_7FF63D3155F0 dq 0FFFFFFFFFFFFFFFFh
seg035:00007FF63D3155F0                                         ; DATA XREF: seg035:off_7FF63D3153F0↑o
seg035:00007FF63D3155F0                                         ; seg035:off_7FF63D315410↑o ...
seg035:00007FF63D3155F8 dq offset sub_7FF63D3122F8
seg035:00007FF63D315600 dq offset sub_7FF63D313B90
```

查看第一个函数：

```cpp
_BYTE *sub_7FF63D3122F8()
{
  _BYTE *BeingDebugged; // rax
  _BYTE *p_sub_7FF63D311648; // [rsp+48h] [rbp-8h]

  BeingDebugged = (_BYTE *)NtCurrentTeb()->ProcessEnvironmentBlock->BeingDebugged;
  if ( !(_BYTE)BeingDebugged )
  {
    qword_7FF63D318140 = off_7FF63D3192F0(1, sub_7FF63D3121C5);
    srand(0x93E71989);
    for ( p_sub_7FF63D311648 = sub_7FF63D311648; ; ++p_sub_7FF63D311648 )
    {
      BeingDebugged = (char *)&loc_7FF63D312648 - 4;
      if ( p_sub_7FF63D311648 >= (_BYTE *)&loc_7FF63D312648 - 4 )
        break;
      if ( *p_sub_7FF63D311648 == 65
        && p_sub_7FF63D311648[1] == 66
        && p_sub_7FF63D311648[2] == 67
        && p_sub_7FF63D311648[3] == 68 )
      {
        BeingDebugged = p_sub_7FF63D311648 + 4;
        qword_7FF63D318150 = (__int64)(p_sub_7FF63D311648 + 4);
        return BeingDebugged;
      }
    }
  }
  return BeingDebugged;
}
```

发现含有反调试，且含有srand，检查其中的函数：

```cpp
__int64 __fastcall sub_7FF63D3121C5(__int64 a1)
{
  __int64 v2; // [rsp+38h] [rbp-28h]
  __int64 v3; // [rsp+40h] [rbp-20h]
  __int64 v4; // [rsp+48h] [rbp-18h]
  _QWORD *v5; // [rsp+50h] [rbp-10h]

  v5 = *(_QWORD **)(a1 + 8);
  if ( **(_DWORD **)a1 != 0xC0000094 )
    return 0;
  if ( _InterlockedCompareExchange((volatile signed __int32 *)&unk_7FF63D318148, 1, 0) )
    return 0;
  v4 = v5[16];
  v3 = v5[17];
  v2 = v5[23];
  if ( !v4 || !v3 || !v2 )
    return 0;
  sub_7FF63D3120C9(v4, v3, v2, v5[24]);
  if ( qword_7FF63D318150 )
    v5[31] = qword_7FF63D318150;
  else
    v5[31] += 2LL;
  return 0xFFFFFFFFLL;
}

unsigned __int64 __fastcall sub_7FF63D3120C9(__int64 a1, __int64 a2, __int64 a3, unsigned __int64 a4)
{
  _DWORD *v4; // rbx
  unsigned __int64 result; // rax
  _BYTE v6[11]; // [rsp+2Dh] [rbp-23h] BYREF
  unsigned __int64 v7; // [rsp+38h] [rbp-18h]
  size_t n3; // [rsp+40h] [rbp-10h]
  unsigned __int64 i; // [rsp+48h] [rbp-8h]

  v7 = (a4 + 2) / 3;
  *(_QWORD *)&v6[3] = a3;
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( i >= v7 )
      break;
    *(_WORD *)v6 = 0;
    v6[2] = 0;
    if ( a4 >= 3 * (i + 1) )
      n3 = 3;
    else
      n3 = a4 - 3 * i;
    memcpy(v6, (const void *)(a1 + 3 * i), n3);
    v4 = (_DWORD *)(4 * i + *(_QWORD *)&v6[3]);
    *v4 = sub_7FF63D3114C9(v6, 3);
  }
  return result;
}

__int64 __fastcall sub_7FF63D3114C9(unsigned __int8 *a1, __int64 n3)
{
  int v2; // ebx
  int i; // [rsp+28h] [rbp-8h]
  unsigned int t; // [rsp+2Ch] [rbp-4h]
  int m; // [rsp+58h] [rbp+28h]

  m = n3;
  t = 114514;
  for ( i = 0; i < m; ++i )
  {
    v2 = (t << 19) + 8 * a1[i] + 19 * t;
    t = v2 + 10 * rand_0();
  }
  return t;
}
```

`0xC0000094` 是除零异常的标识符，说明这里在非调试状态下会注册除以零异常处理的handler，跳转至其中这个每3字节一哈希的加密，之后扫描了垃圾函数 `sub_7FF63D311648`，检索其中的特殊组合[65,66,67,68]并跳转至其后：

```asm
seg035:00007FF63D3120B8 unk_7FF63D3120B8 db  58h ; X            ; CODE XREF: sub_7FF63D311648+A6D↑j
seg035:00007FF63D3120B9 db  41h ; A
seg035:00007FF63D3120BA db  42h ; B
seg035:00007FF63D3120BB db  43h ; C
seg035:00007FF63D3120BC db  44h ; D
seg035:00007FF63D3120BD ; ---------------------------------------------------------------------------
seg035:00007FF63D3120BD mov     eax, 0
seg035:00007FF63D3120C2 add     rsp, 78h
seg035:00007FF63D3120C6 pop     rbx
seg035:00007FF63D3120C7 pop     rbp
seg035:00007FF63D3120C8 retn
```

发现就是跳到了垃圾函数的结尾，那么也就是说程序中的RC4加密初始化以及密钥均是无效的，真正的加密逻辑是哈希运算，编写脚本解密：

```cpp
#include <iostream>
#include <cstdint>
#include <vector>
#include <cstdlib>
#include <cstdio>

const uint32_t expected_hash[16] = {
    0xE70F6EB4,
    0xB7741AEE,
    0x77B3F96C,
    0x3A7D82E4,
    0x0DF89E70,
    0x70C13CEC,
    0x55602656,
    0x94B4BC8A,
    0x43310CE4,
    0x5F357476,
    0xD724D53A,
    0x6DE31BB0,
    0xE5210B0E,
    0xA49BF77A,
    0xCF381F00,
    0x363ED066
};

// Compute hash for 3-byte chunk using pre-recorded rand values
inline uint32_t compute_hash(uint8_t b0, uint8_t b1, uint8_t b2, int r0, int r1, int r2) {
    uint32_t h = 114514;
    h = ((h << 19) + h * 19U + 8U * b0 + 10U * static_cast<uint32_t>(r0));
    h = ((h << 19) + h * 19U + 8U * b1 + 10U * static_cast<uint32_t>(r1));
    h = ((h << 19) + h * 19U + 8U * b2 + 10U * static_cast<uint32_t>(r2));
    return h;
}

int main() {
    const uint32_t SEED = 0x93E71989U;
    std::srand(SEED);

    // Generate the exact 48 rand() values used in encryption
    std::vector<int> rands(48);
    for (int i = 0; i < 48; ++i) {
        rands[i] = std::rand();
    }

    std::cout << "Brute-forcing with ASCII range 32~126 (printable chars only)...\n\n";

    std::vector<char> plaintext;

    for (int group = 0; group < 16; ++group) {
        uint32_t target = expected_hash[group];
        int r0 = rands[group * 3 + 0];
        int r1 = rands[group * 3 + 1];
        int r2 = rands[group * 3 + 2];

        bool found = false;
        for (int a = 32; a <= 126 && !found; ++a) {
            for (int b = 32; b <= 126 && !found; ++b) {
                for (int c = 32; c <= 126; ++c) {
                    if (compute_hash(a, b, c, r0, r1, r2) == target) {
                        plaintext.push_back(static_cast<char>(a));
                        plaintext.push_back(static_cast<char>(b));
                        plaintext.push_back(static_cast<char>(c));
                        std::printf("Group %2d: \"%c%c%c\" (0x%02X 0x%02X 0x%02X)\n",
                                    group, a, b, c, a, b, c);
                        found = true;
                        break;
                    }
                }
            }
        }

        if (!found) {
            std::cerr << "[-] No valid printable ASCII triplet found for group " << group << "\n";
            return 1;
        }
    }

    // Output full result
    std::cout << "\n[+] Decrypted plaintext (" << plaintext.size() << " bytes):\n";
    std::cout << "Raw: ";
    for (char ch : plaintext) {
        std::cout << ch;
    }
    std::cout << "\nHex : ";
    for (char ch : plaintext) {
        std::printf("%02X ", static_cast<unsigned char>(ch));
    }
    std::cout << "\n";

    return 0;
}
```

`flag{C4tcH_th3_rUn71m3_3rr0R_0f_d1vId3D_8Y_z3R0}`
