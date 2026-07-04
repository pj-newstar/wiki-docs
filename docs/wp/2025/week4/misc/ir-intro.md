---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 应急响应：初识

解压压缩包，打开 VMware 右键->扫描虚拟机

![VMware 扫描虚拟机](/assets/images/wp/2025/week4/ir-intro_1.png)

选择刚才解压的目录，点击完成。启动虚拟机，选择我已复制该虚拟机

![启动复制的虚拟机](/assets/images/wp/2025/week4/ir-intro_2.png)

输入密码进入桌面后，打开设置-应用-安装的应用，可以看到安装了 phpstudy

打开 phpstudy，在网站栏，点击管理-打开根目录，可以快速定位网站路径

![phpstudy 网站根目录](/assets/images/wp/2025/week4/ir-intro_3.png)

##### 木马连接密码

我们进入 uploads 文件夹，在查看中，勾选查看隐藏的项目，可以看到\_\_\_.php

![隐藏的 PHP 木马文件](/assets/images/wp/2025/week4/ir-intro_4.png)

打开文件时，发现 windows defender 爆出发现威胁。我们还原后查看文件

![冰蝎木马密钥校验](/assets/images/wp/2025/week4/ir-intro_5.png)

可以看出这是冰蝎马，$key 验证连接密码 md5 值的前 16 位，我们将默认密码计算 md5，可以发现前 16 位一致

得到木马连接密码 `rebeyond`

##### 创建账号工具发布时间

打开 User 目录，在 User 目录下有一个 exe 文件，且 windows defender 再次爆出威胁提示

![创建账号工具文件](/assets/images/wp/2025/week4/ir-intro_6.png)

我们搜索该 exe 的文件名，可以得知这个 exe 程序就是创建账号的工具，在 github 项目中 Releases 得到时间

![GitHub Release 发布时间](/assets/images/wp/2025/week4/ir-intro_7.png)

`2022_01_18`

##### 影子用户密码

这里我使用 `mimikatz` 去获取密码 hash

```plaintext
privilege::debug
lsadump::sam
```

![Mimikatz 导出 SAM Hash](/assets/images/wp/2025/week4/ir-intro_8.png)

得到 Hash NTLM 后，使用 hashcat 进行爆破

![Hashcat 破解 NTLM Hash](/assets/images/wp/2025/week4/ir-intro_9.png)

```bash
hashcat.exe -m 1000 -a 3 7e5c358b43a26bddec105574bee24eef ?a?a?a?a?a?a
hashcat.exe -m 1000 -a 3 7e5c358b43a26bddec105574bee24eef ?a?a?a?a?a?a --show
```

`Ns2025`
