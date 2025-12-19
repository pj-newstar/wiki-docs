---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# strange_login

<Container type='info'>

本题考查简单的 SQL 注入漏洞利用。
</Container>

这道题是一个典型的 SQL 注入挑战。题目描述中的 `1=1` 暗示了我们可以利用逻辑真值来绕过登录验证。我们猜测后台的 SQL 查询语句是：

```php
$sql = "SELECT * FROM users WHERE username = '$user' AND password = '$pass'";
```

这个查询语句将会把用户输入的用户名（`$user`）和密码（`$pass`）直接拼接到 SQL 语句中，没有进行任何过滤或转义，导致 SQL 注入漏洞。

我们的目标是通过注入，使 SQL 查询条件始终为真，从而绕过身份验证。常用的注入载荷是 `1' or 1=1#`，但这里需要根据查询语句的结构进行调整。

## 解题步骤

1. **理解查询结构**：查询语句是`SELECT * FROM users WHERE username = '$user' AND password = '$pass'`。我们需要在用户名或密码字段中输入特殊字符来改变查询逻辑。
2. **选择注入点**：通常，我们在用户名字段进行注入，因为密码字段可能被哈希处理（但这里没有）。假设我们在用户名输入框注入。
3. **构造注入载荷**：

- 如果我们输入用户名：`1' or 1=1#`，密码任意（例如 `123`），那么查询语句会变成：

```sql
SELECT * FROM users WHERE username = '1' or 1=1#' AND password = '123'
```

- 解释：

  - `1'` 闭合了用户名的单引号。
  - `or 1=1` 添加了一个永远为真的条件。
  - `#` 是MySQL中的注释符，会注释掉后面的所有内容，包括 `AND password = '123'`。

- 因此，实际执行的查询是：

```sql
SELECT * FROM users WHERE username = '1' or 1=1
```

这个查询会返回 `users` 表中的所有行，通常登录验证会检查是否有返回结果，如果有就登录成功。

4. **实际尝试**：

- 在用户名输入框输入：`1' or 1=1#`
- 在密码输入框输入任意值（例如 `123`）。
- 点击登录，应该会成功绕过登录。

5. **为什么这样工作**：因为 `1=1` 总是真，所以查询条件永远成立，返回所有用户数据。注释符 `#` 确保了密码检查被忽略，避免了密码错误。
