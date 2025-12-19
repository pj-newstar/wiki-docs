---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# multi-headach3

<Container type='info'>

本题考查对 HTTP 协议及 `robots.txt` 文件的理解与应用。
</Container>

## 前言：给CTF新手的话

这是一道 Web 类型的 CTF 题目，主要考查你对 HTTP 协议和 Web 基础知识的理解。如果你是第一次接触 CTF，不用担心，我会一步步带你理解解题思路。

## 第一步：访问网站，观察页面内容

首先，我们访问题目给出的网站地址。你会看到一个简单的网页，内容大概是这样：

```plaintext
Hello!
Today is 2025/08/28
welcome to my first website!

ROBOTS is protecting this website!
But... Why my head is so painful???!!!
```

### 初步分析

看到这个页面，有几个关键信息需要注意：

1. **"ROBOTS is protecting this website!"** - 这里明确提到了"ROBOTS"
2. **"Why my head is so painful"** - 这里提到了"头痛"，结合题目名称"multi-headach3"

作为CTF新手，你可能会想：什么是ROBOTS？为什么要强调头痛？

## 第二步：了解 robots.txt 文件

### 什么是 robots.txt？

在 Web 开发中，`robots.txt` 是一个特殊的文件，用来告诉搜索引擎爬虫（机器人）哪些页面可以访问，哪些页面不能访问。这个文件通常放在网站的根目录下。

### 如何访问 robots.txt？

在浏览器地址栏中，在网站地址后面加上 `/robots.txt`，比如：

```plaintext
http://目标网站/robots.txt
```

### 动手操作

让我们访问一下这个网站的 `robots.txt` 文件。你会看到：

```plaintext
User-agent: *
Disallow: /hidden.php
```

### 理解 robots.txt 内容

- `User-agent: *` 表示对所有的爬虫生效
- `Disallow: /hidden.php` 表示禁止爬虫访问 `/hidden.php` 这个页面

**重要发现**：网站管理员不想让搜索引擎收录 `hidden.php` 这个页面！

## 第三步：探索隐藏页面

现在我们知道了有一个 `hidden.php` 页面，让我们尝试访问它：

```plaintext
http://目标网站/hidden.php
```

### 奇怪的现象

**最终答案**: 通过访问 `/robots.txt` 发现隐藏页面 `/hidden.php`，然后使用开发者工具查看该页面的HTTP响应头，在 `Fl4g` 头部中找到flag。
