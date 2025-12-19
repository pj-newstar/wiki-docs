---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# EzMyDroid

<Container type='info'>

本题考查 Java 层安卓逆向中 AES 加密的分析与破解。
</Container>

`Jadx-gui` 打开搜索字符串 `Wrong`，定位代码

![alt text](/assets/images/wp/2025/week1/ezmydroid_1.png)

可以看到一个加密调用，和密文比较。

加密如下，看类名可以知道是 `AES` 的 `ECB` 模式，第一个参数是输入的 `flag` 第二个参数是密钥。

```java
String encrypt = AESECBUtils.encrypt(FirstFragment.this.binding.input.getText().toString(), "1145141919810000");
```

密文肯定就是，被 base64 编码过。

```java
encrypt.equals("cTz2pDhl8fRMfkkJXfqs2t8JBsqLkvQZDLYpWjEtkLE=")
```

进入解密网站 [赛博厨子](https://cyberchef.org/)

![alt text](/assets/images/wp/2025/week1/ezmydroid_2.png)
