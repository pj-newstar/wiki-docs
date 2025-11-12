---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 日志分析-不敬者的闯入 

题目中附件给了一个 `access.log`。在线环境打开是抗日战争暨世界反法西斯战争胜利 80 周年的网站。

我们分析 `access.log`，筛选 `状态码 200` 的日志

在日志里可以看到 ip 为 `171.16.20.72` 访问了 `/admin/Webshell.php` 的记录，同时 `/admin/` 也是允许被访问的

![日志分析](/assets/images/wp/2025/week2/chuangru_1.png)

我们访问 `/admin/` 可以发现有目录浏览漏洞，而在里面只有 `Webshell.php`，访问后可以发现连接密码即为 flag