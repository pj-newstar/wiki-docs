---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 流量分析：听声辩位

打开 `blindsql.pcapng`，点击「统计」->「协议分级」，可以看到主要是 TCP、HTTP 协议。

![Wireshark 协议分级统计](/assets/images/wp/2025/week4/traffic-sound-locate_1.png)

在 `Line-based text data` 字段，可以看出是网页源代码。

点击「文件」->「导出对象」->「HTTP」，可以大致查看流量的整体行为。

![导出 HTTP 对象](/assets/images/wp/2025/week4/traffic-sound-locate_2.png)

这里可以明显看出是 SQL 注入流量。

选中一条 HTTP 流量，右键选择「追踪流」->「HTTP Stream」，主面板上的包列表就只会列出本次 HTTP 会话的数据包，同时会在一个单独的窗口中显示 HTTP 流的内容。

在 HTTP 流窗口中，数据通常会以两种颜色显示，红色表示从源地址到目的地址的流量，蓝色则表示从目的地址到源地址的流量。

![HTTP Stream 查看回显](/assets/images/wp/2025/week4/traffic-sound-locate_3.png)

如果中文没有正常显示，将「显示为 ASCII」改成 UTF-8 编码即可。

逐条查看 HTTP 流，可以发现回显只有两种。对 GET 请求做 URL 解码后，可以确认这是二分法盲注：如果真实值大于盲注数值，则条件为真；如果真实值小于盲注数值，则条件为假。

先过滤出 HTTP 流量，然后点击「文件」->「导出分组解析结果」->「AS CSV」，选择已显示的数据并保存。

![导出分组解析结果](/assets/images/wp/2025/week4/traffic-sound-locate_4.png)

![保存 CSV 文件](/assets/images/wp/2025/week4/traffic-sound-locate_5.png)

在 CSV 中可以清晰地看到流量的简略内容。

通过 `Source: 171.16.20.89` --> `Destination: 171.16.20.55` 的 `Length`，可以在 Wireshark 中分析出 `1091` 为真，`1124` 为假。

通过对注入 SQL 语句的大致分析，可以看出最终注入目的是从 `newstar2025` 库中 `week4` 表的 `value` 字段中提取数据。

搜索 `newstar2025.week4` 并查找全部匹配项，可以定位到最开始的注入语句进行分析，也可以从最后倒序分析。

这里有一个小技巧：已知相关流量从第 1378 条开始，删除其他无关内容后，另存为 TXT 格式，大致整理成如下形式：

![整理盲注流量文本](/assets/images/wp/2025/week4/traffic-sound-locate_6.png)

使用 Notepad++ 打开，选择全部内容，再点击「插件」->「MIME Tools」->「URL Decode」。

这样可以得到更清晰的内容。

![URL Decode 后的注入语句](/assets/images/wp/2025/week4/traffic-sound-locate_7.png)

为了便于分析，还可以将 `1091` 全部替换为真，`1124` 替换为假。

最后将分析出的数值汇总并解码即可得到 FLAG。

**如何分析？**

以第一位为例，`> 100` 为真，`> 102` 为假，`> 101` 为真，因此第一位为 `102`。
