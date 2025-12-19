---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# Puzzle

<Container type='info'>

本题考查 IDA 软件中字符串搜索的使用。
</Container>

> [!INFO]
> 这道题主要目的是让新生知道 IDA 的一些常见操作。

用 IDA 打开附件之后，单击键盘 <kbd>F5</kbd> 反编译出 main 函数

![main 函数](/assets/images/wp/2025/week1/puzzle_1.png)

首先观察到有一个函数 `Puzzle_Challenge`，点进去可以看到：

![函数内容](/assets/images/wp/2025/week1/puzzle_2.png)

这里面是 flag 的第一部分即`Do_Y0u_`

接着继续往后看，根据提示 `functions have strange names`，在 `function` 列表中可以看到一个 `Like_7his_Jig` 函数和一个 `Its_about_part3` 函数。

点进 `Like_7his_Jig` 函数 ：

![Like_7his_Jig 函数内容](/assets/images/wp/2025/week1/puzzle_3.png)

可以知道函数名字即为 flag 的第二部分 `Like_7his_Jig`

点进 `Its_about_part3` 函数：

![第三部分](/assets/images/wp/2025/week1/puzzle_4.png)

有一个异或 `0xad`，密文在 `encrypted_array` 中，<kbd>⇧ Shift</kbd><kbd>e</kbd> 提取出来为：`0xDE, 0xED, 0xDA, 0xF2, 0xDD, 0xD8, 0xD7, 0xD7`

解密：

```py
enc = [0xDE, 0xED, 0xDA, 0xF2, 0xDD, 0xD8, 0xD7, 0xD7]
part3 = ""
for i in range(len(enc)):
    part3 += chr(enc[i] ^ 0xad)
print(part3)
#s@w_puzz
```

得到 flag 的第三部分即 `s@w_puzz`
<kbd>⇧ shift</kbd><kbd>F12</kbd> 查看字符串注意到可疑的字符串：`1e_Gam3`，点进去看到：

![第四部分](/assets/images/wp/2025/week1/puzzle_5.png)

所以 flag 的第四部分即 `1e_Gam3`

最后将四个部分拼接起来再包裹上 `flag{}`，最终的 flag 即为 `flag{Do_Y0u_Like_7his_Jigs@w_puzz1e_Gam3}`
