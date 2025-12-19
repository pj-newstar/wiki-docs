---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# DD 加速器

<Container type='info'>

本题考查命令拼接的 RCE 利用。
</Container>

打开题目，我们得到这样一个页面。

![dd-accelerator_1](/assets/images/wp/2025/week2/dd-accelerator_1.png)
![dd-accelerator_2](/assets/images/wp/2025/week2/dd-accelerator_2.png)

点击开始，可以看到系统 ping 命令产生的输出。我们可以猜测，这个功能就是执行了类似下面的**系统命令**实现的。

```bash
ping -c 1 127.0.0.1
```

那么，我们想要攻入这台服务器，获取 flag，是不是可以想一想，有没有方法一行执行多个命令？有的兄弟，有的。

## 介绍一下 `|` 和 `&`:

### `|` — 管道（pipe）

作用：**把左边命令的标准输出<span data-desc>（stdout）</span>连接到右边命令的标准输入<span data-desc>（stdin）</span>。**

### `&` — 后台运行符<span data-desc>（ampersand）</span>

作用：把命令放到**后台执行**，shell 立即返回提示符，继续执行后面的命令或接受输入。

### 还有两个逻辑处理

```bash
cmd1 && cmd2 # cmd1 成功 (exit 0) 时才执行 cmd2
cmd1 || cmd2 # cmd1 失败时执行 cmd2
```

这些都可以让一行执行多个命令。

回到这题来：
我们的输入填在了 IP 的位置，也就是说可以往后添加一个管道符 `|`，就能注入命令了。

```bash
ping -c 1 127.0.0.1 | ls
```

> 这题的非预期是用 `env`，出题人失误忘记清环境变量了

这题的 flag 存在于一个隐藏文件夹。所以我们要用 `ls -la <路径>` 的方式查看。

![dd-accelerator_3](/assets/images/wp/2025/week2/dd-accelerator_3.png)

尝试 `cat`：限制了长度。那该怎么办呢？这个时候就该用通配符 `*` 了。

通配符是 **shell 用来匹配文件名的特殊符号**。

当你输入一个命令<span data-desc>（如 ls \*.txt ）</span>时，shell 在执行命令之前会先展开通配符，把它替换成所有匹配的文件名，再把结果传给命令。

例如：

```sh
ls *.txt
```

→ Shell 会自动展开成:<span data-desc>（假设存在 a.txt b.txt notes.txt ）</span>

```sh
ls a.txt b.txt notes.txt
```

然后执行这个命令。

所以我们只需要 `cat .<首字母>*/flag` 就能读取到 flag 了。
