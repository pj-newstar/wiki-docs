---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 宇宙的中心是 PHP

<Container type='info'>

本题考查前端源代码分析、PHP 函数理解及绕过技巧。
</Container>

进入后是一个前端小页面，会发现直接<kbd>F12</kbd>或者右键是没办法打开开发者工具的，这里提供几种办法查看源代码

1. 先开启开发者工具再访问网站
2. 使用 <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>I</kbd>
3. 手动在 url 处输入 `view-source:` 前缀

然后就可以看到下一关的入口提示（这里以第三个方法为例）

![level1 示例](/assets/images/wp/2025/week1/center-php_1.png)

```plaintext
<!-- 你还是找到了......这片黑暗的秘密 -->
<!-- s3kret.php -->
```

访问这个 php 文件可以看到代码

```php
<?php
highlight_file(__FILE__);
include "flag.php";
if(isset($_POST['newstar2025'])){
    $answer = $_POST['newstar2025'];
    if(intval($answer)!=47&&intval($answer,0)==47){
        echo $flag;
    }else{
        echo "你还未参透奥秘";
    }
}
```

这里是想考察一下 newstar 们遇到未知函数的检索能力

先来分析一下函数逻辑

首先代码要求 POST 名为 newstar2025 的参数，随后将参数以两种方式传入 intval 函数，我们可以到 PHP 的文档里查阅一下这个函数的作用和用法 [PHP: intval Manual](https://www.php.net/manual/zh/function.intval.php)

![php-intval](/assets/images/wp/2025/week1/center-php_2.png)

可以看到，`intval($value)` 和 `intval($value, 0)` 的区别在于后者会根据输入字符串的格式自动匹配进制，而前者则是默认的十进制

因此本体的绕过方式就非常明朗了，只需要传入一个其它进制的数字 `47` 即可，以下 POST 传参均可

```plaintext
newstar2025=057
newstar2025=0x2f
newstar2025=0b101111
```
