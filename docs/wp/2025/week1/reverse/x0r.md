---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# X0r

<Container type='info'>

本题考查异或加密的逆向分析。
</Container>

加密逻辑都在 `main` 函数中：

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char Str2[32]; // [rsp+20h] [rbp-60h] BYREF
  _BYTE v5[16]; // [rsp+40h] [rbp-40h]
  char Str[36]; // [rsp+50h] [rbp-30h] BYREF
  int n24; // [rsp+74h] [rbp-Ch]
  int n24_2; // [rsp+78h] [rbp-8h]
  int n24_1; // [rsp+7Ch] [rbp-4h]

  _main();
  puts("Please input your flag: ");
  scanf("%25s", Str);
  n24 = strlen(Str);
  if ( n24 == 24 )
  {
    for ( n24_1 = 0; n24_1 < n24; ++n24_1 )
    {
      if ( n24_1 % 3 )
      {
        if ( n24_1 % 3 == 1 )
          Str[n24_1] ^= 0x11u;
        else
          Str[n24_1] ^= 0x45u;
      }
      else
      {
        Str[n24_1] ^= 0x14u;
      }
    }
    v5[0] = 19;
    v5[1] = 19;
    v5[2] = 81;
    for ( n24_2 = 0; n24_2 < n24; ++n24_2 )
      Str[n24_2] ^= v5[n24_2 % 3];
    strcpy(Str2, "anu`ym7wKLl$P]v3q%D]lHpi");
    if ( !strcmp(Str, Str2) )
      puts("Right flag!");
    else
      puts("Wrong flag!");
    return 0;
  }
  else
  {
    puts("Wrong flag length!");
    return 0;
  }
}
```

有两次加密操作，第一次是遍历字符串，根据索引对 3 取模的不同情况来进行异或，第二次是按周期异或数组，最后进行结果比较。

解密就从后往前逆回去，exp 如下：

```python
enc = b"anu`ym7wKLl$P]v3q%D]lHpi"
v5 = [19, 19, 81]
temp = bytearray(enc)
for i in range(len(enc)):
    temp[i] = temp[i] ^ v5[i % 3]

for i in range(len(temp)):
    if i % 3 == 0:
        temp[i] ^= 0x14
    if i % 3 == 1:
        temp[i] ^= 0x11
    if i % 3 == 2:
        temp[i] ^= 0x45

flag = temp.decode()
print(flag)
```

最后得到的 flag 为 `flag{y0u_Kn0W_b4s1C_xOr}`
