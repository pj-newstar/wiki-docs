---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 日志分析：不敬者的闯入

<Container type='info'>

本题考查日志分析和漏洞排查的基本能力。
</Container>

题目中附件给了一个 `access.log`。在线环境打开是一个网站。

我们分析 `access.log`，筛选 `状态码 200` 的日志

在日志里可以看到 ip 为 `171.16.20.72` 访问了 `/admin/Webshell.php` 的记录，同时 `/admin/` 也是允许被访问的

![日志分析](/assets/images/wp/2025/week2/intruder_1.png)

我们访问 `/admin/` 可以发现有目录浏览漏洞，而在里面只有 `Webshell.php`，访问后可以发现连接密码即为 flag.
