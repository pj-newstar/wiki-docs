---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 白帽小 K 的挑战（1）

<Container type='info'>

本题考查信息泄露和文件上传漏洞。
</Container>

本题提供了一个看似非常像音乐播放器的内容，但是我们可以上传属于自己的音乐。

你可以使用 Wappalyzer 插件来查看本网页使用的服务类型，

![Image](/assets/images/wp/2025/week2/white-k-1_1.png)

通过前端的分析可以得到一个隐藏的接口，这个接口并没有在网页中有实际作用，但是在后端应该是实现了这个接口的，我们可以访问这个接口看看。

![Image](/assets/images/wp/2025/week2/white-k-1_2.png)

我们可以尝试访问这里的网页，或者直接在控制台中调用这个 `fetchload` 函数试试。

![Image](/assets/images/wp/2025/week2/white-k-1_3.png)

这里发现 load 下来的内容使用某种方法进行进行解析，我们可以直接使用抓包工具进行访问。

这里我们可以上传一个一句话木马文件上去试试。

![Image](/assets/images/wp/2025/week2/white-k-1_4.png)

然后用 `onload` 接口进行访问。

![Image](/assets/images/wp/2025/week2/white-k-1_5.png)

成功进行命令执行了。然后就是正常的远程 RCE 执行，我们直接 `cat /flag` 就行了。

事实上，这道题目的后端中，使用了 `include` 来执行 php 代码，并且将执行结果返回。在更远的未来，你会见到很多有关 php 的上传恶意文件后加载恶意文件进行 RCE 的操作。
