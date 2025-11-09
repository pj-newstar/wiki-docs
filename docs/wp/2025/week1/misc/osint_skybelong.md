---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# OSINT-天空belong

> [!info]
> 本题内容为 OSINT 部分的图片寻址类问题。

对于新手第一次接触该类型的问题，可以跳转链接进行了解 [OSINT - Hello CTF](https://hello-ctf.com/hc-misc/osint/) 以便于解出此题。

下载完附件，进行解压，我们可以得到一张图片，图片中可以在机翼上看到一个标号 `B-7198`。

![](/assets/images/wp/2025/week1/osint_skybelong_1.png)

然后查看图片的 EXIF 信息，拍摄时间是 `2025/08/17 15:03:47`	拍摄设备的制造商为 `Xiaomi`

在前面链接里了解的 [flightaware](https://www.flightaware.com/) 中搜索这个标号，应该是飞机的注册号。

![](/assets/images/wp/2025/week1/osint_skybelong_2.png)

我们往下查看对应时间的历史航班。

![]( /assets/images/wp/2025/week1/osint_skybelong_3.png )

可以得到飞机的航班号为 `UQ3574`。

![]( /assets/images/wp/2025/week1/osint_skybelong_4.png )

在下面的航线图中，我们调整到拍摄时间，可以发现该航班经过的省会城市有且只有**武汉市**。

![]( /assets/images/wp/2025/week1/osint_skybelong_5.png )

 所以答案是 `flag{UQ3574_武汉市_Xiaomi}` 