---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# OSINT：威胁情报

<Container type='info'>

本题考查开源情报收集与分析能力。
</Container>

题目给了一个恶意情报的 hash 值，我们可以借助公开的威胁情报中心来完成此题。

这里我使用的是[奇安信威胁情报中心](https://ti.qianxin.com/)和 [VirusTotal](https://www.virustotal.com/gui/home/search)。

题目需要我们找到关于恶意文件的`apt 组织名称`、`通信 C2 服务器域名`、`恶意文件编译时间(年-月-日)`

## APT 组织名称

奇安信威胁情报中心中，在首页我们就可以直接得到攻击组织的名称

![攻击组织名称](/assets/images/wp/2025/week2/qingbao_1.png)

VirusTotal：

在 Family labels 中显示多个标签，我们可以在社区进行进一步的确定。

![攻击组织名称](/assets/images/wp/2025/week2/qingbao_2.png)

在社区中，通过其他检测人员的确认确定 apt 家族名称。

![apt 家族名称](/assets/images/wp/2025/week2/qingbao_3.png)

## 通信 C2 服务器域名

奇安信威胁情报中心：

我们需要点击`样本分析详情`得到动态运行的详细内容。

![样本分析详情](/assets/images/wp/2025/week2/qingbao_4.png)

在`网络行为`栏，我们可以得到域名

![获得域名](/assets/images/wp/2025/week2/qingbao_5.png)

VirusTotal：

在 `BEHAVIOR` - `Activity Summary` - `Memory Patten Domains` 中可以确定它的域名。

![VirusTotal](/assets/images/wp/2025/week2/qingbao_6.png)

## 恶意文件编译时间（年 - 月 - 日）

奇安信威胁情报中心：

在`基本信息` - `Exiftool 文件元数据` - `TimeStamp` 代表恶意文件的编译时间

![编译时间](/assets/images/wp/2025/week2/qingbao_7.png)

VirusTotal：

在`DETAILS` - `History` - `Create Time` 可以得知编译时间

![编译时间](/assets/images/wp/2025/week2/qingbao_8.png)
