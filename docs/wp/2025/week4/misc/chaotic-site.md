---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 混乱的网站

本题打开后是一个 CTF 介绍网站。查看源码可以发现 `function.js`，其中 misc 模块会调用 `_return()`。

![源码中的 function.js](/assets/images/wp/2025/week4/chaotic-site_1.png)

将混淆后的 JavaScript 做十六进制解码和格式化，可以看到 `_pick` 会选取数组中的第三个元素并做 Base64 解码。实际执行 `_return()` 时，会跳转到 `https://newstar.games/`，同时把 `window["js_very_good"]` 赋给 `window["$"]["$"]`。因此 `js_very_good` 很可能是 FLAG 的一部分。

题目描述提到网站还没搭建完成，并且页面中存在注册入口。继续进行目录扫描，可以发现 `www.zip` 备份文件。

![目录扫描发现 www.zip](/assets/images/wp/2025/week4/chaotic-site_2.png)

下载备份后，在 `dashboard.php` 中发现一段 Gzip + Base64 的混淆内容。

![dashboard.php 混淆内容](/assets/images/wp/2025/week4/chaotic-site_3.png)

将执行函数改成输出函数后，可以得到一层 `o00o0o` 风格的嵌套混淆。

![解出嵌套混淆代码](/assets/images/wp/2025/week4/chaotic-site_4.png)

继续把后续 Base64 内容取出解码，再替换执行函数为输出函数。这里建议输出时加上 `htmlspecialchars`，否则解出的 PHP 代码会直接执行，页面上看不到源码。

最终解出的逻辑会持续写入一个后门文件，其中关键条件是 `flag2` 参数的 MD5 值等于：

```plaintext
87c298a56e0caa355872ab47db11e06c
```

查得 `flag2` 为 `ns2025`。

![触发后门得到 FLAG](/assets/images/wp/2025/week4/chaotic-site_5.png)

结合前面得到的 `js_very_good`，最终 FLAG 为：

```plaintext
FLAG{js_very_good_ns2025}
```
