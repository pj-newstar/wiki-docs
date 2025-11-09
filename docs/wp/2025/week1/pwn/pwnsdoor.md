---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# pwn's door

> [!INFO]
> 本题考查的是 nc 的使用

在点击下发赛题后会有如下内容

![容器界面](/assets/images/wp/2025/week1/pwnsdoor_1.png)

```bash
nc ip port
```

ip 和 port 是远程地址，在安装了 netcat 的情况下，在终端指向上述命令就能连接上靶机。如何安装 netcat 可以查看

![nc 连接](/assets/images/wp/2025/week1/pwnsdoor_2.png)

程序分析的话，就用 ida pro 打开，找到 `main` 函数。

![IDA 分析](/assets/images/wp/2025/week1/pwnsdoor_3.png)

发现输入一个数字然后比较输入 ，是 `7038329` 就能通过了

![alt text](/assets/images/wp/2025/week1/pwnsdoor_4.png)

这样就是成功了，然后键入 `cat flag` 就能获取到 flag 了。

![alt text](/assets/images/wp/2025/week1/pwnsdoor_5.png)