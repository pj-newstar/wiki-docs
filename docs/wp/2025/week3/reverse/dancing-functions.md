---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# Dancing Functions

主函数：

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int v4; // [rsp+2Ch] [rbp-14h]
  void *Block; // [rsp+30h] [rbp-10h]
  char *Str1; // [rsp+38h] [rbp-8h]

  sub_140002060(argc, argv, envp);
  SetConsoleOutputCP(0xFDE9u);
  sub_140003130("Input your message to encrypt: ");
  Str1 = (char *)malloc(0x10000u);
  sub_1400030E0("%s");
  if ( (unsigned int)sub_140001BCD(Str1) )
  {
    Block = (void *)KeyGen();
    sub_140001C42(Block, Str1);
    v4 = strcmp(Str1, Str2);                    // ".sBtQ=0JEhC#sbw=Q-Y*3h-PGpcvZ9SbU+9F5tH96e>-5hMF"
    sub_140003130("The Chemical form of your encrypted message is: ");
    Dont_Look_At_Me_I_Am_Just_A_Print_Function(Str1);
    if ( v4 )
      sub_140003130("Wrong FLAG!\n");
    else
      sub_140003130("Right FLAG!\n");
    free(Block);
  }
  else
  {
    sub_140003130("Wrong FLAG format!The input should be in ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz012345678"
                  "9!#$%&*+-.<=>?@_{|}~\n");
  }
  free(Str1);
  return 0;
}
```

有一个长度为81的字符表，约束了输入的字符集，还有一个没什么用的打印函数

keygen是一个函数指针，静态指向一个函数：

```cpp
_BYTE *sub_1400018D2()
{
  srand(0x11451419u);
  do
  {
    do
      n11 = rand() % 81;
    while ( n11 <= 11 );
  }
  while ( n11 > 16 );
  v1 = malloc(n11 + 1);
  for ( n11_1 = 0; n11_1 < n11; ++n11_1 )
    v1[n11_1] = Str[rand() % 81];
  v1[n11] = 0;
  return v1;
}
```

但keygen还有多个xref，其中有一个函数可以在main之前执行：

```cpp
__int64 I_Can_Run_Before_Main()
{
  if ( IsDebuggerPresent() )
    KeyGen = sub_1400016DD;
  else
    KeyGen = (__int64 (__fastcall *)())lpAddress_;
  if ( (unsigned int)pthread_create(&v1, 0, p_sub_1400019C9, 0) )
    sub_140002C60(1);
  return pthread_detach(v1);
}
```

检测了调试器，切换了keygen到一个地址，查看这个地址：

```asm
.text:00000001400017DA
.text:00000001400017DA ; ---------------------------------------------------------------------------
.text:00000001400017DB lpAddress_      db 99h, 84h, 45h, 29h, 84h
.text:00000001400017DB                                         ; DATA XREF: p_sub_1400019C9+C↓o
.text:00000001400017DB                                         ; I_Can_Run_Before_Main:loc_140001AC5↓o ...
.text:00000001400017E0                 dq 0CCDD89D875FC204Fh, 34890BCCCCD0C724h, 0CCD73B24CCCCCCCCh
.text:00000001400017F8                 dq 0A5840EAF840E45CCh, 240D84D5847C310Ch, 350D1D45CF340DECh
.text:0000000140001810                 dq 814734894504E5D3h, 4CDCF2C0D044534h, 0CDCCCCCCCC09C041h
.text:0000000140001828                 dq 0B14F3499450EE504h, 0DC34B14F0CB2C734h, 3489475C7427CEB2h
.text:0000000140001840                 dq 0D45845484CD0C4Fh, 894584CCCCD03724h, 0CCCCCCCC30890B3Ch
.text:0000000140001858                 dq 45CCCCD75D249B27h, 310CA5840DAF840Dh, 45EC240D84D5847Ch
.text:0000000140001870                 dq 340D0445CF360D0Eh, 0CF2C0D1C450EE5D3h, 0CCCCCC09D8411CCDh
.text:0000000140001888                 dq 8406450DE51CCDCCh, 0CCFBABD941840EAFh, 308947DCC07AC3CCh
.text:00000001400018A0                 dq 0CD843C9947845484h, 30894FDC4406451Ch, 0B03489F7308947CDh
.text:00000001400018B8                 dq 478454843489476Dh, 0CCCC0A1CCD843C99h, 0FC084F843C894784h
.text:00000001400018D0                 db 91h, 0Fh
.text:00000001400018D2
.text:00000001400018D2 ; =============== S U B R O U T I N E =======================================
.text:00000001400018D2
```

发现无法解析，再查看这个地址的xref：

```cpp
__int64 p_sub_1400019C9()
{
  HANDLE hProcess; // rax
  DWORD flOldProtect; // [rsp+2Ch] [rbp-24h] BYREF
  SIZE_T dwSize; // [rsp+30h] [rbp-20h]
  __int64 (__fastcall *p_sub_1400018D2)(); // [rsp+38h] [rbp-18h]
  LPVOID lpAddress; // [rsp+40h] [rbp-10h]
  SIZE_T dwSize_1; // [rsp+48h] [rbp-8h]

  lpAddress = lpAddress_;
  p_sub_1400018D2 = sub_1400018D2;
  dwSize = (char *)sub_1400018D2 - (char *)lpAddress_;
  if ( VirtualProtect(lpAddress_, (char *)sub_1400018D2 - (char *)lpAddress_, 0x40u, &flOldProtect) )
  {
    for ( dwSize_1 = 0; dwSize_1 < dwSize; ++dwSize_1 )
      *((_BYTE *)lpAddress + dwSize_1) ^= 0xCCu;
    VirtualProtect(lpAddress, dwSize, flOldProtect, &flOldProtect);
    hProcess = GetCurrentProcess();
    FlushInstructionCache(hProcess, lpAddress, dwSize);
  }
  return 0;
}
```

可以发现其在运行过程中被xor了0xCC以进行自解密，恢复解密后的函数：

```cpp
unsigned char* TrueKeyGen() {
    srand(0x114514);
    int len = 0;
    while (1) { len = rand() % 81; if (12 <= len && len <= 16) { break;}}
    unsigned char* key = (unsigned char*)malloc(len + 1);
    for (int index = 0; index < len; index++) { key[index] = alphabet[rand() % 81];}
    key[len] = '\0';
    return key;
}
```

查看加密函数：

```cpp
void sub_140001C42(unsigned char* key, unsigned char* message) {
    // 初始化密钥
    int keyLen = strlen((const char*)key);
    int messageLen = strlen((const char*)message);
	Init(message, messageLen);

    // 初始化 S 盒
    int sboxSize = strlen(alphabet);
    unsigned char sbox[sboxSize];
    for (int i = 0; i < sboxSize; i++) {
        sbox[i] = 80 - i; // 0~80
    }

    // KSA
    unsigned char t = 0;
    for (int j = 0; j < sboxSize; j++) {
        t = (t + sbox[j] + key[j % keyLen]) % sboxSize;
        // 交换 sbox[j] 和 sbox[t]
        unsigned char temp = sbox[j];
        sbox[j] = sbox[t];
        sbox[t] = temp;
    }

    // PRGA
    int m = 0, n = 0;
    for (int k = 0; k < messageLen; k++) {
        m = (m ^ n) % sboxSize;
        n = (n + sbox[m]) % sboxSize;
        // 交换 sbox[m] 和 sbox[n]
        unsigned char temp = sbox[m];
        sbox[m] = sbox[n];
        sbox[n] = temp;
        message[k] = alphabet[(message[k] + sbox[(sbox[m] ^ sbox[n]) % sboxSize]) % sboxSize];
    }
}
```

是一个类RC4函数，只不过空间只有81，编写脚本解密：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static const char alphabet[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!#$%&*+-.<=>?@_{|}~";

int char_to_index(unsigned char c) {
    for (int i = 0; i < 81; i++) {
        if (alphabet[i] == c)
            return i;
    }
    return -1;
}

unsigned char* TrueKeyGen() {
    srand(0x114514);
    int len = 0;
    while (1) {
        len = rand() % 81;
        if (12 <= len && len <= 16) {
            break;
        }
    }
    unsigned char* key = (unsigned char*)malloc(len + 1);
    for (int index = 0; index < len; index++) {
        key[index] = alphabet[rand() % 81];
    }
    key[len] = '\0';
    return key;
}

int main() {
    unsigned char ciphertext[] = ".sBtQ=0JEhC#sbw=Q-Y*3h-PGpcvZ9SbU+9F5tH96e>-5hMF";
    int cipher_len = strlen((char*)ciphertext);
    unsigned char* key = TrueKeyGen();
    int keyLen = strlen((char*)key);
    printf("key: %s\n", key);
    int sboxSize = 81;
    unsigned char sbox[81];
    for (int i = 0; i < sboxSize; i++) {
        sbox[i] = 80 - i;
    }

    unsigned char t = 0;
    for (int j = 0; j < sboxSize; j++) {
        t = (t + sbox[j] + key[j % keyLen]) % sboxSize;
        unsigned char temp = sbox[j];
        sbox[j] = sbox[t];
        sbox[t] = temp;
    }

    int m = 0, n = 0;
    unsigned char* plaintext = (unsigned char*)malloc(cipher_len + 1);

    for (int k = 0; k < cipher_len; k++) {
        m = (m ^ n) % sboxSize;
        n = (n + sbox[m]) % sboxSize;
        unsigned char temp = sbox[m];
        sbox[m] = sbox[n];
        sbox[n] = temp;
        unsigned char keystream = sbox[(sbox[m] ^ sbox[n]) % sboxSize];
        int cipher_idx = char_to_index(ciphertext[k]);
        if (cipher_idx == -1) {
            fprintf(stderr, "Invalid cipher char: %c\n", ciphertext[k]);
            exit(1);
        }
        int plain_idx = (cipher_idx - keystream + sboxSize) % sboxSize;
        plaintext[k] = alphabet[plain_idx];
    }
    plaintext[cipher_len] = '\0';

    printf("Plaintext: %s\n", plaintext);

    free(key);
    free(plaintext);
    return 0;
}
```

`flag{D4nc1Ng_K3yG3N_fUnct10n5_wItH_r4nD_4Nd_5Mc}`
