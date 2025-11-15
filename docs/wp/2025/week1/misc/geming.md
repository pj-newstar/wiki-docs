---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 我不要革命失败

<Container type='info'>

本题考查 Windows 操作系统出现蓝屏的日志分析。
</Container>

直接下载 [windbg](https://learn.microsoft.com/zh-cn/windows-hardware/drivers/debugger/) 然后打开。

依照提示或者直接点击超链接，就会开始分析蓝屏日志。

![开始分析](/assets/images/wp/2025/week1/geming_1.png)

最开始的这一条，即为崩溃类型<span data-desc>（如果蓝屏过的话，这就是蓝屏下面的那行终止代码）</span>

![崩溃类型](/assets/images/wp/2025/week1/geming_2.png)

![蓝屏](/assets/images/wp/2025/week1/geming_3.png)

然后可以从下图位置找到故障进程。

![alt text](/assets/images/wp/2025/week1/geming_4.png)

:::warning
如果使用 windows 自带的 notepad 打开文本文档，默认缩放下会看不到下划线的，记得放大。
:::
