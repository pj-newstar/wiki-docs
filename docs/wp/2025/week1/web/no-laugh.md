---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 别笑，你也过不了第二关

<Container type='info'>

本题考查前端源代码分析、HTTP 协议参数 HTTP 请求报文的发送。
</Container>

先打开游戏随便玩玩，发现第二关目标分数 1000000，无法正常完成。

但是这是一道 web 题，所以我们开始收集信息 <kbd>Ctrl</kbd> + <kbd>U</kbd> 打开源代码，发现游戏的主要逻辑在最后的 `script` 标签中，我们找到判断通关的逻辑：

```js
if (score >= targetScores[currentLevel]) {
  // 如果达成目标分数
  alert(`恭喜通过第 ${currentLevel + 1} 关！得分: ${score}`);
  currentLevel++;
  if (currentLevel < targetScores.length) {
    // 下一关
    resetLevel(currentLevel);
    startGame();
  } else {
    // 全部通关
    gameEnded = true;
    const formData = new URLSearchParams();
    formData.append("score", score);
    // 提交分数到服务器，fetch /flag.php 获取 flag
    fetch("/flag.php", {
      method: "POST",
      headers: {
        "Content-Type": "application/x-www-form-urlencoded",
      },
      body: formData.toString(),
    })
      .then((res) => res.text())
      .then((data) => {
        alert("服务器返回:\n" + data);
      })
      .catch((err) => {
        alert("请求失败: " + err);
      });
  }
}
```

于是我们便有了下面这些方法

法一：在浏览器控制台修改前端分数

<kbd>F12</kbd> 打开控制台，输入以下代码：

```js
score = 1000000; // 直接把分数改成目标值
currentLevel = 1; // 直接到最后一关
endLevel(); // 调用关卡结束函数提交
```

法二：模拟POST请求

```sh
curl -X POST http://目标服务器/flag.php -d "score=1000000"
```

法三：通过抓包工具修改

但是要注意

```plaintext
POST请求
Content-Type: application/x-www-form-urlencoded

请求体：
score=100000
```

举例：

```plaintext
POST /flag.php HTTP/1.1
Host: 47.104.154.99:22221
Content-Type: application/x-www-form-urlencoded
.............
Cookie: session=xxx
Connection: keep-alive
Content-Length: 11

score=10001
```
