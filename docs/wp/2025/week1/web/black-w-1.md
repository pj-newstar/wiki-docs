---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 黑客小 W 的故事（1）

<Container type='info'>

本题考查 HTTP 请求协议及抓包工具的使用。
</Container>

欢迎来到骨钉大师的考验。

![起始页](/assets/images/wp/2025/week1/black-w-1_1.png)

## 第一关

![挑战目标](/assets/images/wp/2025/week1/black-w-1_2.png)

本关需要你连续点击这只虫子，最后获得 800 吉欧来到达下一关，可是在中途会被突如其来的古神给打乱阵脚，从而失去所有的吉欧。

本关的提示中给出，你可以使用**抓包**的方法来处理这个过程

![第一关提示](/assets/images/wp/2025/week1/black-w-1_3.png)

我们可以下载 [yakit](https://www.yaklang.com/) 来帮助我们实现抓包的操作。

打开 yakit 之后，我们可以随便创建一个临时项目来处理这次的题目。

![yakit 使用教程](/assets/images/wp/2025/week1/black-w-1_4.png)

打开之后可以点击启动劫持。

![yakit 使用教程](/assets/images/wp/2025/week1/black-w-1_5.png)

你有若干种方法来使用 yakit 的劫持进行抓包，比较方便的一种方法是使用免配置启动。

![yakit 使用教程](/assets/images/wp/2025/week1/black-w-1_6.png)

在使用免配置启动之后，yakit 会自动打开一个浏览器，并且抓取和记录这个浏览器里面的所有访问流量，非常像我们在使用 chrome 开发工具中的网络工具。

![yakit 使用教程](/assets/images/wp/2025/week1/black-w-1_7.png)

等我们访问这个网页之后，就能在软件界面中看到所有的流量信息了，我们可以点击其中一个看看详细的请求包与响应包的内容。

![yakit 使用教程](/assets/images/wp/2025/week1/black-w-1_8.png)

当然，抓包操作的话，我们可以点击左上角的手动劫持，然后点击一下网页中的虫子，看看我们的网页往后端发送了什么内容。

![yakit 使用教程](/assets/images/wp/2025/week1/black-w-1_9.png)

所有前后端进行交互的内容都会被这里拦截住。根据上面的提示，我们可以一次性多打几只虫子，看来这里的数量就是通过 `count` 参数实现的。我们直接修改这个参数为 100000 试试。

![修改参数](/assets/images/wp/2025/week1/black-w-1_10.png)

于是我们便通过了第一关。

## 第二关

本题中需要你和蘑菇先生交流，你可以选择使用浏览器的 hackbar 插件完成本题。你也可以继续使用 yakit 的抓包放包操作来完成。

提示中提到了你需要在 `GET` 参数中将 `shipin` 参数设为 `mogubaozi`。

此时再提到与蘑菇先生说话，他就会让你用 POST 参数告诉他你想要什么，根据前面的提示，向蘑菇先生提及骨钉这个事情。

> [!info]
> 如果使用 hackbar 传参数，不能直接填入 `guding`，需要 `guding=1` 这样的写法。否则 hackbar 不会将你的内容作为 body 传到后端中。

你可以继续拦截这个请求，并且将请求手动修改成 `POST`，然后在请求头中手动填入 `Content-Type: application/x-www-form-urlencoded` 以及 `Content-Length: 123`，这两个参数分别表示了你在使用 `POST` 方法时，你的 Body 的处理编码方式以及 Body 长度。其中 `Content-Length` 会被 yakit 自动修复。

![自定义请求包](/assets/images/wp/2025/week1/black-w-1_11.png)

但是在这之后我们需要用 `DELETE` 方法帮忙蘑菇先生清除掉他身上的虫子。

与 `POST` 方法类似，我们修改 `POST` 方法为 `DELETE` 方法，然后在 body 中填入 `chongzi` 就可以了。

在这之后我们再重新使用 `POST` 方法提交一次 `guding` 就能够去往下一关了。

如果你在这里使用了 Fuzz 工具或者 BurpSuite 中的 repeater 工具进行操作，那么请在前往下一关前修改自己的 Cookie 为新的 Cookie。

> [!warning]
> 此处的 DELETE 参数的使用方法并不符合协议要求，仅供出题使用。

## 第三关

第三关中，提示让我们修改 `User-Agent` 头来表明自己的身份，这里我们不能够仅仅填入 CycloneSlash，还需要填入 CycloneSlash 的版本信息，也就是使用 User-Agent 的正式格式来进行请求。

请求头中填入 `User-Agent: CycloneSlash/1.0`，会发现席奥觉得我们的剑技太慢了，其实是在说我们的剑技版本太低了，需要填入更高的版本，这道题目需要填入大于等于 2.0 的版本。

然后下一关就会让我们填入冲刺斩，以相同的规则填入就可以了，这里的最低版本要求为 5.0。

![UA 要求](/assets/images/wp/2025/week1/black-w-1_12.png)

## 结束

最后到达最后一关就能够直接获得 flag 了。~~本来这里还有一关的，后来觉得太麻烦，就算了。~~
