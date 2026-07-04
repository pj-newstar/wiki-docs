---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 小的明？题问

## 分析

漏洞在下面这个地方

```cpp

    memset(currrnt_account, 0, sizeof(user));
}

//account.c 40
int account_delete(user *current_account, user *input_account)
{
    if(memcmp(current_account, input_account, sizeof(user)))
        return -1;

    int idx = find_account(input_account->username);

    memset(&users[idx], 0, sizeof(user));
    memcpy(&users[idx], &users[idx + 1], sizeof(user) * (user_counts - idx - 1));
    account_logout(current_account);

    user_counts--;
    return 0;
// 在这个地方，你可以开两个 nc 端口，同时登录两个账号
// 然后再把他们都 delete 掉
// 这样 user_counts 就会等于 0，find_user 就找不到 root 账号
// 你就可以注册你自己的 root 帐号了
}

int account_rename(user *current_account, user *old_account, user *new_account)
{
```

另外一个漏洞是在 `add` 的时候程序会把 ` ` 和 `\n` 替换成 `\0`

而 `get_flag` 用的是 `strcmp` 进行比较

只要重命名为 `root 1`，就会被替换成 `root\01`，并通过 `get_flag` 的条件，直接获得 FLAG。

本题无需编写 EXP，使用 nc 交互即可完成利用。
