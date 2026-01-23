---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

WIP

# 被玩坏的 AI

1. 用dirsearch.py去扫发现有个robots.txt文件

然后访问robots.txt

...

...

这里其实就是提示了有个RPO漏洞

2.然后我们访问呢find.php，并且尝试进行利用RPO构造payload

`/RPO/..%2ffind.php`

这里我们得到了password

`🤖 哔......系统消息： yours password：@pwdisadmin`

2. 之后我们输入密码

发现好像还没有解决，又给了个

`不过，要拿到真正的 flag 还需要 Admin 。`

3通过随便输入发现这个要高权限Admin，同时也以机器人口吻，介绍了它名为X，这里算是个提示需要用X自定义标签

然后在网页源码中还有个提示

` // Hint: Admin is Admin only,but Are you Admin?`

这就是想要得到flag，就需要满足X-Admin\:Admin

通过输入

TestUA%0d%0aX-Admin：Admin

进行CRLF注入得到flag
