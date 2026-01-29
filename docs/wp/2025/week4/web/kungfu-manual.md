---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 武功秘籍

## 前言

<Container type='info'>

本题主要考察对于 CVE 的搜查能力，并将其运用在具体的实战场景中
</Container>

![image-20251030201842810](/assets/images/wp/2025/week4/kungfu/image-20251030201842810.png)

看题目描述可以看到很多有效信息，如 CMS、CVE 等等，这时候就可以了解 CVE 是什么

![image-20251030205841812](/assets/images/wp/2025/week4/kungfu/image-20251030205841812.png)

除了 CVE 之外，国外的 NVD 和国内的 CNVD, CNNVD 也是相同的，遇到漏洞的时候记得可以上网搜一下有没有对应的 CVE，再搜搜有没有对应的 PoC，如果有 PoC 那这个漏洞就可以轻松攻破。

![image-20251030210743979](/assets/images/wp/2025/week4/kungfu/image-20251030210743979.png)

进入题目可以先看到这个，通过观察发现这是一个名叫 `dcrcms` 的 CMS 系统，上网搜一下这一个系统有没有漏洞

![image-20251031151907312](/assets/images/wp/2025/week4/kungfu/image-20251031151907312.png)

好的，很快就可以看出来，随便选择一篇博客，然后按博客所写的方法进行漏洞渗透就好了

## 以下是正式 WP

进入靶机后访问 `/dcr/login.htm`，发现登录界面，<kbd>F12</kbd> 打开可以看到注释

![image-20251031152456572](/assets/images/wp/2025/week4/kungfu/image-20251031152456572.png)

说明要弱密码爆破，爆破用 burpsuite 就行，有其他的爆破工具也行，总之最后的账户密码为 admin/admin

![image-20251031152708845](/assets/images/wp/2025/week4/kungfu/image-20251031152708845.png)

有些人可能会被验证码吓到，认为爆破不行，实际上这个验证码是存在 session 中的，只要 session 不变这个验证码就不会变，所以可以放心大胆的爆破

![image-20251031153612826](/assets/images/wp/2025/week4/kungfu/image-20251031153612826.png)

进入后台后点击添加新闻类

![image-20251031153657686](/assets/images/wp/2025/week4/kungfu/image-20251031153657686.png)

再添加新闻

![image-20251031153742591](/assets/images/wp/2025/week4/kungfu/image-20251031153742591.png)

![image-20251031153805506](/assets/images/wp/2025/week4/kungfu/image-20251031153805506.png)

这里是它的文件上传的漏洞点，抓包上传一句话木马

![image-20251031153931124](/assets/images/wp/2025/week4/kungfu/image-20251031153931124.png)

这里改成 image/jpeg 再放行，成功后去系统管理/文件管理器，文件目录：根目录 `/uploads/news/2025_10_31`

![image-20251031154337626](/assets/images/wp/2025/week4/kungfu/image-20251031154337626.png)

然后执行任意命令即可

![image-20251031154445875](/assets/images/wp/2025/week4/kungfu/image-20251031154445875.png)

## 总结

这是一个非常简单的一个 CVE 题目，所需要具备的是信息检索能力的知识储备能力，当发现这个系统有 CVE 漏洞的时候就可以上网去查询 CVE 信息以及 PoC，这道题当有了 PoC 之后就是一个简单的文件上传题
