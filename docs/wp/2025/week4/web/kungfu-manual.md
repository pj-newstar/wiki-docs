---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 武功秘籍

<Container type='info'>

本题主要考查 CVE 信息检索与漏洞利用能力。
</Container>

## 前置分析

![题目描述中的 CMS 和 CVE 信息](/assets/images/wp/2025/week4/kungfu-manual_1.png)

题目描述提供了 CMS、CVE 等有效信息，可以据此检索漏洞资料。

![CVE 资料检索结果](/assets/images/wp/2025/week4/kungfu-manual_2.png)

除 CVE 之外，NVD、CNVD、CNNVD 等漏洞库也可以作为检索入口。遇到已知漏洞时，可以先确认是否存在对应 CVE，再检索是否有公开 PoC。

![DcrCMS 题目页面](/assets/images/wp/2025/week4/kungfu-manual_3.png)

进入题目后，可以通过页面特征确认这是 DcrCMS 系统，随后检索该系统的公开漏洞。

![DcrCMS 漏洞利用资料](/assets/images/wp/2025/week4/kungfu-manual_4.png)

检索后可以找到相关漏洞利用文章，按其中步骤进行验证即可。

## 解题过程

进入靶机后访问 `/dcr/login.htm`，发现登录界面，按 <kbd>F12</kbd> 可以看到注释。

![登录页面注释提示](/assets/images/wp/2025/week4/kungfu-manual_5.png)

注释提示需要进行弱口令爆破，可以使用 Burp Suite 或其他爆破工具。最终得到账号密码为 `admin/admin`。

![Burp 爆破弱口令](/assets/images/wp/2025/week4/kungfu-manual_6.png)

验证码存储在 Session 中，只要保持同一 Session，验证码就不会变化，因此可以复用该 Session 进行爆破。

![验证码 Session 复用](/assets/images/wp/2025/week4/kungfu-manual_7.png)

进入后台后点击添加新闻类。

![后台添加新闻类](/assets/images/wp/2025/week4/kungfu-manual_8.png)

再添加新闻。

![后台添加新闻](/assets/images/wp/2025/week4/kungfu-manual_9.png)

![新闻内容编辑页面](/assets/images/wp/2025/week4/kungfu-manual_10.png)

这里是文件上传漏洞点，抓包上传一句话木马。

![文件上传抓包修改](/assets/images/wp/2025/week4/kungfu-manual_11.png)

将 Content-Type 改为 `image/jpeg` 后放行。上传成功后，进入「系统管理」-「文件管理器」，文件目录为根目录 `/uploads/news/2025_10_31`。

![后台文件管理器定位上传文件](/assets/images/wp/2025/week4/kungfu-manual_12.png)

然后执行任意命令即可。

![WebShell 执行命令](/assets/images/wp/2025/week4/kungfu-manual_13.png)

## 总结

本题的关键在于根据 CMS 信息检索对应 CVE 和公开 PoC。确认利用路径后，后续步骤本质上是一个文件上传漏洞利用。
