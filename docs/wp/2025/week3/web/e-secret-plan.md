---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# 小 E 的秘密计划

<Container type='info'>

本题主要考查备份泄露、Git 泄露和 DS_Store 泄露的利用。
</Container>

结合页面提示 `先找到网站备份文件` 使用 dirsearch 进行备份扫描，下载 `www.zip` 文件

根据提示我们知道网站使用 [git](https://git-scm.com/install/) 进行版本管理。如果你不熟悉 Git，可以前往 <https://www.runoob.com/git/git-tutorial.html> 进一步学习。

`git reflog` 会显示所有引用（HEAD、分支等）的移动历史，包括切换、合并和删除操作。

![git reflog 删除分支记录](/assets/images/wp/2025/week3/e-secret-plan_1.png)

我们可以看到有一个叫做 `test` 的分支被删除了，可以优先恢复并排查。

我们复制 reflog 每一行前 7 位 Hash，使用下列命令还原。

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

![DS_Store 文件解析结果](/assets/images/wp/2025/week3/e-secret-plan_2.png)

得到 FLAG 地址，访问后获得最终 FLAG。
