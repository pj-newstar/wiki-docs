---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 欧皇的生日

<Container type='info'>

本题考查生日攻击。
</Container>

题目脚本比较简单，就不分析了。

简单来说我们需要提交两个数 $x_1,x_2$ 使得两数哈希值相同，如果不相同，会返回这两数的哈希值。这里的哈希函数是一个自定义的模二次式。

参数： $m=2^{22},t=5000$
也就是说我们要在 $[0,2^{22}-1]$ 范围内找到一对**碰撞**，我们有 5000 次尝试次数。

理论计算可以知道，5000 次绰绰有余。

实际交互时，我们可以把我们提交的数和返回的哈希值用字典存起来，直到发生碰撞。

> [!NOTE] 如何交互？
>
> 先安装 pwn 包。
>
> ```bash
> pip install pwn
> ```
>
> 然后编写脚本：
>
> ```python
> from pwn import remote
> HOST,PORT = "??.??.??.??",?????
> io=remote(HOST,PORT)  #连接靶机，端口。这里 io 可以看成一个交互的实例
> print(io.recvline())  #打印靶机输出的提示信息，类型是 byte
> io.sendline(f"{x1} {x2}")  #向靶机发送数字。注意如果你提交的不是 byte 格式，程序会提醒你（
>
> ```

> [!NOTE]- 非预期
> 理论上你可以把$a,b,c$求出来，然后本地找碰撞。
