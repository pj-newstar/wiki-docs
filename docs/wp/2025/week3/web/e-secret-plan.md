---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 小 E 的秘密计划

> 本题主要考察对于各种泄露的利用，包括备份泄露，git泄露，DS_Store泄露

结合页面提示 `先找到网站备份文件` 使用 dirsearch 进行备份扫描，下载 `www.zip` 文件

根据提示我们知道网站使用 [git](https://git-scm.com/install/) 进行版本管理, 如果你不熟悉 git，可以前往 <https://www.runoob.com/git/git-tutorial.html> 进一步学习。

`git reflog` 会显示所有引用（HEAD、分支等）的移动历史，包括切换、合并和删除操作。

![reflog](/assets/images/wp/2025/week3/web_e_reflog.png)

我们可以看到有一个叫做 `test` 的分支被删除了，可以优先恢复并排查。

我们复制 reflog 每一行前的7位 hash，使用下列命令还原

```bash
git branch test 353b98f
git checkout test
```

此时 `user.php` 泄露了明文账密

```php
<?php

function getUserData() {
    return [
        'username' => 'admin',
        'password' => 'f75cc3eb-21e0-4713-9c30-998a8edb13de'
    ];
}
```

使用此账密登录 `/public-555edc76-9621-4997-86b9-01483a50293e` 进入 dashboard。
之后结合提示 `小E拿 mac 写的这段代码`，扫描发现 [.DS_Store](https://baike.baidu.com/item/.DS_Store/3395876) 文件。

下载 `.DS_Store` 文件后，尝试利用 [ds_store_exp](https://github.com/lijiejie/ds_store_exp) 工具还原信息。

![dsstore](/assets/images/wp/2025/week3/web_e_dsstore.png)

得到 flag 地址，访问获得最终 flag。
