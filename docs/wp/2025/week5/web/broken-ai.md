---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 被玩坏的 AI

## 信息搜集

dirsearch 扫描发现有个 `robots.txt` 文件，内容如下

```txt
User-agent: *
Allow: /find.php

Disallow: /RPO/

```

访问 `find.php`，控制台提示：

这里可没有，你想要的password，试着找找是不是在其他的find.php中？

访问 `/RPO/`，没有返回有效数据，但是经过搜索，RPO 是一种漏洞的名称。

> 这种漏洞全称为 Relative Path Overwrite, 漏洞存在的核心是关于服务器和客户端的解析差异，我们很早之前就知道对于锚点所可能造成的安全问题，
> 比如在挖掘 SSRF 相关漏洞如果在后台简单的判断后缀必须为 .png 我们可以通过 #.png 来达到一个成功访问的 bypass.
> 首先 RPO 存在的条件为开启了URL重写之后，当我们这样访问 /index.php/1234 网站时，服务器角度会把子元素识别为参数，
> 而浏览器其实是无法分辨哪一部分是参数，整个路径都会被识别为一个路径。
> (引用来源：<https://xz.aliyun.com/news/14450>)

我们经过多次尝试，得出正确 payload

`/RPO/..%2ffind.php`

这里利用了浏览器将 `..%2ffind.php` 识别为路径，而服务器却将 `%2f` 重写为 `/`，从而达成了 LFI 获得了正确的路径。

在控制台我们得到：`🤖 哔......系统消息： yours password：@pwdisadmin`

## 利用

我们回到主页输入密码，发现问题还没有解决，

`不过，要拿到真正的 flag 还需要 Admin 。`

通过随便输入发现这个要高权限 Admin，同时也以机器人口吻，介绍了它名为 X，这里算是个提示需要用 X 自定义标签

然后在网页源码中还有个提示

`// Hint: Admin is Admin only,but Are you Admin?`

这就是想要得到 flag，就需要满足 `X-Admin\:Admin`

通过输入

```txt
TestUA%0d%0aX-Admin: Admin
```

进行 CRLF 注入得到 flag
