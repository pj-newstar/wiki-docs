---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 日志分析：盲辨海豚

下载压缩包并解压，可以发现其中包含 `blindsql.log` 日志文件。

打开文件，可以看出这是典型的布尔盲注日志。

![布尔盲注日志](/assets/images/wp/2025/week3/log-blind-dolphin_1.png)

观察响应长度，不难看出如果正确，返回长度为 `6`

编写脚本，提取返回长度为 6 的请求日志：

```python
with open('blindsql.log', 'r') as f:
    lines = f.readlines()
logtxt = []
for i in range(len(lines)):
    log = lines[i]
    if '200 6' in log:
        logtxt.append(log)
with open('log.txt', 'w', encoding='utf-8') as file:
    file.writelines(logtxt)
```

根据 SQL 注入语句可知，从第 21 条记录开始按位爆破 FLAG。

![按位爆破请求](/assets/images/wp/2025/week3/log-blind-dolphin_2.png)

这里既可以手动提取并转换得到 FLAG，也可以使用正则表达式提取 FLAG 对应的 ASCII 值，再进行转码。

这里使用正则表达式进行演示：

```python
import re
from urllib.parse import unquote
with open('log.txt', 'r') as f:
    lines = f.readlines()
FLAG = ''
for i in range(len(lines)):
    txt = lines[i]
    logs = unquote(txt)
    pattern = r"ascii\(substr\(\(select FLAG from sqli\.FLAG\),(\d+),1\)\)='(\d+)'"
    matches = re.findall(pattern, logs)
    for position, ascii_val in sorted(matches, key=lambda x: int(x[0])):
        FLAG += ''.join(chr(int(ascii_val)))
        print(FLAG)
```

运行后即可得到 FLAG。
