---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 应急响应：把你 mikumiku 掉（1）

我们查看 tomcat 的版本信息，可以得到，版本号为 `9.0.98`

![Windows 日志文件](/assets/images/wp/2025/week5/mikumiku-1_1.png)

我们搜索 `tomcat9.0.98 cve` 可以搜索到编号为 `CVE-2025-24813` 的历史漏洞

![筛选异常登录事件](/assets/images/wp/2025/week5/mikumiku-1_2.png)

结合日志，可以确定使用的 cve

```plaintext
cat localhost_access_log.2025-10-17.txt
```

![定位 RDP 登录 IP](/assets/images/wp/2025/week5/mikumiku-1_3.png)

**CVE-2025-24813**
