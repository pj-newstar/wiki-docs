---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 白帽小 K 的故事（2）

这是一道 SQL 盲注题目，跟随流程，查看 hint，给了两点提示

1. 后端查询逻辑是 `SELECT 1 from Terra.animal WHERE name = '$name'`
2. 要使用盲注

这提示我们

1. 可以利用 `'` 闭合达成 SQL 注入
2. 注入后响应不包含 sql 语句执行结果或原始报错

简单手动 fuzz 一下 waf，发现只过滤了空格（抓包返回 `{"status":"error","message":"Invalid characters detected"}`），我们用括号绕过就行。

我们构造 `amiya'AND(${payload})#`，由于用户 amiya 存在，`WHERE name = 'amiya'` 为真，此时 `payload` 的真假决定了表达式的返回，如果 `payload` 为真，应当返回正常登录的结果，反之，则会返回用户不存在。这就符合了**布尔盲注**的条件。

由于手动盲注耗时长，我们需要编写脚本去自动完成盲注，通常我们会使用二分搜索算法和多线程来优化盲注速度。这里给出我的解题脚本：

<https://github.com/nick-cjyx9/blind-sqli/blob/master/examples/newstar.ts>
