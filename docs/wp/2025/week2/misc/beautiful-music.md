---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 美妙的音乐

<Container type='info'>

本题考查 midi 音频隐写的分析。
</Container>

midi 音频隐写，听声音没听出来什么异常，那么有可能是不发声的音符隐写，用 [miditrail](https://www.yknk.org/miditrail/en/download/)<span data-desc>（也可以是其他工具，只要能查看 midi 的音符就行）</span>打开，可以观察到：

![miditrail 界面](/assets/images/wp/2025/week2/beautiful-music_1.png)

于是最后的 flag 就是 `flag{thi5_1S_m1Di_5tEG0}`。
