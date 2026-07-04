---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 区块链：INTbug

在 `usePoints()` 函数的 `unchecked` 代码块中：

```plaintext
unchecked {
    totalPoints -= points;
    userSpentPoints[msg.sender] -= points;
}
```

**初始状态**：每个用户调用 `addPoints()` 或 `usePoints()` 时，如果 `userSpentPoints[msg.sender]` 为 0，会被设置为 1000。

如果满足 uncheckd 下方判断，则会触发解锁

```plaintext
if (userSpentPoints[msg.sender] > 1000) {
	unlocked[msg.sender] = true;
}
```

也就是说，只要使用的点数大于 1000，就可以触发解锁

我们在 `addPsoints` 中输入大于 1000 的数，`addPoints` 执行后再进行 `usePoints` 大于 1000 的数即可

![整数溢出调用链](/assets/images/wp/2025/week5/intbug-chain_1.png)

点击 `getFlag`，即可得到 FLAG
