---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 应急响应：把你 mikumiku 掉（3）

进入 `/home/mikuu`，可以看到恶意程序和被加密的 `flag.miku`。

![恶意程序与加密文件](/assets/images/wp/2025/week5/mikumiku-3_1.png)

使用 FTP 将文件传到本地，并用 IDA 打开 `mikumikud` 进行分析。

![mikumikud 反编译分析](/assets/images/wp/2025/week5/mikumiku-3_2.png)

可以看出这是一个 AES 加密程序，分析 `key` 和 IV 后即可编写解密脚本。

```plaintext
AES_set_encrypt_key(v13, 128LL, v10)
```

可以得出 `v13` 数组为 `key`。需要注意小端序低字节在前，因此 `key` 为：

```plaintext
12 34 56 78 9A BC DE F0 11 22 33 44 55 66 77 88
```

IV 来自 `ptr` 和 `v12` 变量：

```asm
unsigned __int64 ptr; // [rsp+930h] [rbp-50h]
__int64 v12; // [rsp+938h] [rbp-48h]
ptr = 0xEF40511401811919LL;
v12 = 0x1032547698BADCFELL;
```

在加密过程中，IV 被写入文件开头。

```plaintext
fwrite(&ptr, 1uLL, 0x10uLL, s);  // 写入16字节：ptr(8字节) + v12(8字节)
```

知道 `key` 和 IV 后，编写脚本运行即可还原 FLAG。

```python
from Crypto.Cipher import AES

key = bytes.fromhex('123456789ABCDEF01122334455667788')
with open('flag.miku', 'rb') as f:
    iv = f.read(16)
    ciphertext = f.read()

cipher = AES.new(key, AES.MODE_CBC, iv)
plaintext = cipher.decrypt(ciphertext)
pad_len = plaintext[-1]
print(plaintext)
```
