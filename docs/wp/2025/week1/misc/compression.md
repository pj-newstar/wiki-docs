---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# Misc 城邦：压缩术

<Container type='info'>

题考查压缩包暴力破解、伪加密、明文攻击等压缩包相关知识点。
</Container>

本题为压缩包隐写常见考点，在写这题之前，可以查阅关于压缩包&隐写的相关知识点。

我们先读题目描述，题目描述中说`请挑战者们通过6位密码门开始挑战吧！(要想使用压缩术，请先念咒语"abcd...xyz0123...789")`可以得知第一关是密码暴力破解，且是只由数字和小写字母组成的6位密码。

我们可以使用相关工具，如 `ARCHPR`、`Passware kit` 等，也可以自己写脚本。

这里我使用的是ARCHPR，选择密码暴力范围。

![image-20251105011158940](/assets/images/wp/2025/week1/compression_1.png)

点击开始，我们就得到了第一关压缩包加密的密码：`ns2025`。

![image-20251105011303164](/assets/images/wp/2025/week1/compression_2.png)

解压后得到 `tips.txt` 和 `bkcrack.zip` <span data-desc>（这里框住，后面要考）</span>。

在 `tips.txt` 中提到`（下一扇门明明没有密码，为什么还是要输入密码呢？）`我们可以得知第二关为伪加密。

在此之前，我们可以了解一下 zip 文件的文件结构。

**ZIP文件头 <span data-desc>(Local file header)</span>**

| 字段名称                    | 长度(byte) | 说明                                                       |
| --------------------------- | ---------- | ---------------------------------------------------------- |
| Local file header signature | 4          | 文件头标识，固定值 `50 4B 03 04`                           |
| Version needed to extract   | 2          | 解压文件所需的 ZIP 最低版本                                |
| General purpose bit flag    | 2          | 通用位标志，通常只需要考虑当 `bit 0` 为 1 时表示文件被加密 |
| Compression method          | 2          | 压缩方式，当值为 0x0000 时表示无压缩                       |
| Last mod file time          | 2          | 文件最后修改时间，以 standard MS-DOS 格式编码              |
| Last mod file date          | 2          | 文件最后修改日期                                           |
| CRC-32                      | 4          | 未压缩数据的 CRC32                                         |
| Compressed size             | 4          | 压缩后的大小，单位为 byte                                  |
| Uncompressed size           | 4          | 未压缩的大小                                               |
| File name length            | 2          | 文件名长度                                                 |
| Extra field length          | 2          | 扩展区域长度                                               |
| File name                   | N          | 文件名                                                     |
| Extra field                 | N          | 扩展区域                                                   |

**核心目录 <span data-desc>(Central directory)</span>**

| 字段名称                                | 长度(byte) | 说明                                                       |
| --------------------------------------- | ---------- | ---------------------------------------------------------- |
| Central directory file header signature | 4          | 中心目录文件头标识，固定值 `50 4B 01 02`                   |
| Version made by                         | 2          | 压缩所用的 ZIP 版本                                        |
| Version needed to extract               | 2          | 解压文件所需的 ZIP 最低版本                                |
| General purpose bit flag                | 2          | 通用位标志，通常只需要考虑当 `bit 0` 为 1 时表示文件被加密 |
| Compression method                      | 2          | 压缩方式，当值为 0x0000 时表示无压缩                       |
| Last mod file time                      | 2          | 文件最后修改时间                                           |
| Last mod file date                      | 2          | 文件最后修改日期                                           |
| CRC-32                                  | 4          | 未压缩数据的 CRC32                                         |
| Compressed size                         | 4          | 压缩后的大小                                               |
| Uncompressed size                       | 4          | 未压缩的大小                                               |
| File name length                        | 2          | 文件名长度                                                 |
| Extra field length                      | 2          | 扩展区域长度                                               |
| File comment length                     | 2          | 文件注释长度                                               |
| Disk number start                       | 2          | 文件开始位置所在的磁盘编号                                 |
| Internal file attributes                | 2          | 内部文件属性                                               |
| External file attributes                | 4          | 外部文件属性                                               |
| Relative offset of local header         | 4          | 本地文件头的相对偏移                                       |
| File name                               | N          | 文件名                                                     |
| Extra field                             | N          | 扩展区域                                                   |
| File comment                            | N          | 文件注释                                                   |

按照上述的文件结构，我们可以得知在`General purpose bit flag`其二进制的最低位为 1，表示文件被标记为加密， 要解决伪加密问题，只需将该标志位修改为 `00 00`，即关闭加密标志。

![image-20251106195219287](/assets/images/wp/2025/week1/compression_3.png)

将 `09 00` 改为 `00 00`。

![image-20251106195250231](/assets/images/wp/2025/week1/compression_4.png)

改完后保存，这事我们就可以发现压缩包就没有显示加密了，我们继续解压缩，得到了 `key.txt` 和 `flag.zip`。

在 `flag.zip` 里面同样有 `key.txt`，结合之前的 bkcrack 的名字，我们不难猜出第三关是压缩包明文攻击。

为了验证是否可以进行明文攻击，我们将 `key.txt` 进行与 `flag.zip` 相同方式的压缩，可以发现他们的 CRC32 校验值一样，则可以进行明文攻击。

![image-20251106205414193](/assets/images/wp/2025/week1/compression_5.png)

接下来我们使用 bkcrack 对 `flag.zip` 进行明文攻击，在此之前我们先了解一下相关参数。

参数解释：

```
-C 加密压缩包
-c 提取的密文部分
-p 提取的明文部分
```

![image-20251106212840738](/assets/images/wp/2025/week1/compression_6.png)

然后我们可以用 key 删除密码，在同一目录下生成一个具有相同内容但不加密的 `flag2.zip`

![image-20251106213042105](/assets/images/wp/2025/week1/compression_7.png)

打开 `flag2.zip` 就可以得到flag了
