---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# pwn's door

<Container type='info'>

本题考查 nc 的使用。
</Container>

在点击下发赛题后会有如下内容：

![容器界面](/assets/images/wp/2025/week1/pwn-door_1.png)

这表示题目需要使用 nc 进行访问，命令格式为：

```bash
nc ip port
```

其中 `ip` 和 `port` 分别是访问 IP（也可以是域名）和端口，在安装了 netcat 的情况下，在终端指向上述命令就能连接上靶机。关于安装 netcat，可参见 [Netcat 使用指南](/learn/nc)。

如上图所示，只需要在终端输入 `nc 8.147.132.32 39543` 即可访问题目环境。具体的访问地址不同选手不一致。

题目给了一个附件，需要进行程序分析，使用用 IDA Pro 打开，会看到如图所示的界面：

![IDA 界面](/assets/images/wp/2025/week1/pwn-door_2.png)

按下 <kbd>F5</kbd> 可以查看反编译后的 C 代码。在左侧「Function name」中找到 `main` 函数并查看反编译结果。

![IDA 分析](/assets/images/wp/2025/week1/pwn-door_3.png)

代码大意为：用户输入一个数字，存入 `v4` 变量，如果 `v4` 等于 `7038329`，则执行 `system("/bin/sh")`，进入 shell.

附件是 Linux 操作系统中的可执行文件格式 ELF，在本地 Linux 环境中运行程序，我们输入 `7038329` 并回车，进入 shell，运行 `whoami` 成功。

![本地运行](/assets/images/wp/2025/week1/pwn-door_4.png)

连接远程环境，输入 `7038329` 进入 shell 并运行 `cat flag` 即可获取 FLAG.

![远程运行](/assets/images/wp/2025/week1/pwn-door_5.png)
