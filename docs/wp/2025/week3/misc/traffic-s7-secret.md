---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 流量分析：S7 的秘密

这是一道工业流量协议的题目，在写本题之前，可以先了解一下 `s7comm` 协议报文的结构

![S7Comm 协议报文结构](/assets/images/wp/2025/week3/traffic-s7-secret_1.png)

附录 7：S7Parameter->Item->Transport size 常见值

![Transport size 常见值](/assets/images/wp/2025/week3/traffic-s7-secret_2.png)

![Transport size 枚举说明](/assets/images/wp/2025/week3/traffic-s7-secret_3.png)

附录 8：S7Parameter->Item->Area 常见值

![Area 常见值](/assets/images/wp/2025/week3/traffic-s7-secret_4.png)

附录 9：S7Data->Item->Return code 已知枚举值

![Return code 枚举值](/assets/images/wp/2025/week3/traffic-s7-secret_5.png)

而在流量里面，对应结构可以观察到 `s7comm.resp.data` 字段是写入的值，`s7comm.param.item.address.byte` 字段影响值的顺序位置。

我们按照顺序排列 `s7comm.resp.data` 字段后解码即可得到 FLAG
