---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 前有文字，所以搜索很有用

<Container type='info'>

本题考查 Base64 编码、零宽字符隐写、SNOW 隐写、Brainfuck 编程语言、摩斯电码、字频统计等杂项内容的综合应用。
</Container>

## Track 1

misc 手们必须要注意**文件名**的信息，因为这里往往都是出题人给予提示的地方。

这一题的文件名称是「fl4g 已经被挤在中间了」。

有做过题目的肯定已经有点头绪，但是不急，我们继续再看一下 txt 文件内容。

文本内容是很多的「零宽字符」的信息，而且还能看到文字后面还有些奇怪的方格。

直接上网搜索零宽字符，其实就能够找到这样的隐写，以及一些线上的网站了，比如[零宽字符在线处理](https://330k.github.io/misc_tools/unicode_steganography.html)。

当然，txt 文档为什么只写了这几种零宽字符？因为就只用了这几种，所以你写脚本也是可以解决的。

零宽字符解密出来之后是一串奇怪的字符 `ZmxhZ3t5b3Vf`，实际上还是直接搜索一下就知道了，`Zmxh` 这一眼就是 flag 前缀嘛，base64 编码是肯定会接触到的。

解决也很简单，线上工具很多，[cyberchef](https://cyberchef.org/)也可以，或者你用[随波逐流](http://150.158.159.129/bo_ctfcode.html)一把梭也可以。

本 track 的结果就是 `flag{you_`。

## Track 2

这一 Track 有两个文件，一个 docx，一个 txt。

关键词想必就是「fxxk」和「brain」了，直接搜索一下，就能搜到 [brainfuck](https://baike.baidu.com/item/Brainfuck/1152785) 这种语言，然后就可以搜索[相应的工具](https://www.splitbrain.org/services/ook)。

再看文本内容，`here's key`，说明这还是某个东西的 key，不是 flag 本身，总之先解密出来得到 key 为：`brainfuckisgooooood`。

得到了 key 之后再回头看 docx 文档，文件名和里面内容的题目都是「咏雪」，既然整个题目都是文本隐写，那就继续搜索跟**雪**有关的隐写。

关键词搜索 **雪 隐写**之后，就能找到 SNOW 隐写的工具。

把 docx 文档内的包含文本后大量空白部分的文本复制到 txt 里面用工具解密即可获得第二段 flag。

```shell
snow.exe -p brainfuckisgooooood input.txt output.txt
```

得到一串人人都知道的是摩斯电码的东西，cyberchef 就好了，最后得到 `0V3RC4ME_`。

## Track 3

这一 Track 还是只有 txt，名称仍然是提示，「谁多谁少，一算便知」，结合 txt 文件里面毫无顺序的文本，其实就是想要做题人去把里面的字符按照多与少的顺序统计出来，按照顺序排列，就是 flag。

上网搜索字频统计，也有[相应的工具](https://uutool.cn/str-statistics/)

最后按照从多到少的顺序就是 flag 最后一部分 `cH@1LenG3s}`

组合起来得到 `flag{you_0V3RC4ME_cH@1LenG3s}`。
