---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 应急响应：把你 mikumiku 掉（2）

结合上题的日志，可以得知 `mikuu.jsp`，我们查看 `mikuu.jsp`

![事件日志中的命令执行](/assets/images/wp/2025/week5/mikumiku-2_1.png)

可以分析出该木马连接密码为 `miiikuuu`

我们查看 `/etc/shadow` 文件，可以看到一个名为 `mikuu` 的用户

![发现恶意程序路径](/assets/images/wp/2025/week5/mikumiku-2_2.png)

可以从 `$y` 中得知他是 `yescrypt` 密码

该类型的密码无法使用 `hashcat`，而 `john` 的效率太低，我们在github上可以看到 [yescrypt_crack](https://github.com/cyclone-github/yescrypt_crack/releases/tag/v0.2.0) 项目

我们配完 go 环境，装依赖，再 go bulid，根据 tips，我们生成由 miku 四个字符构成的密码本

```bash
./yescrypt_crack.exe -h hash.txt -w ezdict.txt  -t 16 -s 10
```

爆破得到 miiiku
