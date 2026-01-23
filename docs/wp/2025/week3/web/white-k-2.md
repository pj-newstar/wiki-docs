---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 白帽小 K 的故事（2）

直接看 hint，给了两点提示

1. 后端查询逻辑是 `SELECT 1 from Terra.animal WHERE name = '$name'`
2. 要使用盲注

简单手动测试一下 waf，发现过滤了空格（抓包返回 `{"status":"error","message":"Invalid characters detected"}`），我们用括号绕过就行

这里给出我的解题脚本 <https://github.com/nick-cjyx9/blind-sqli/blob/master/examples/newstar.ts>
