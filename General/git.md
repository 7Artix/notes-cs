切换分支:

```shell
git checkout <branch_name>
git switch <branch_name>
```

新建并切换到新分支:

```shell
git checkout -b feature/user-system
```

推送本地分支到远程仓库, 并关联:

```shell
git push -u origin feature/login
```

查看本地分支:

```shell
git branch
```

查看远程分支:

```shell
git branch -r
```

查看所有分支 (本地+远程):

```shell
git branch -a
```

查看各个分支最后提交信息:

```shell
git branch -v
```

删除本地分支:

```shell
git branch -d <branch_name>
```

合并指定分支到当前分支:

```shell
git merge <branch_name>
```


查看当前状态:

```shell
git status
```

一次新功能开发 (个人项目):

```shell
git checkout -b feature/user-system # 创建并切换到新分支
git add . # 添加所有修改
git status # 查看文件修改
git commit -m "feat: complete user management system"
git pull --rebase origin main # 从远程仓库更新主分支 pull=fetch+merge

# 团队协作时, 永远不要直接推送到主分支
git checkout main # 切换回主分支
git merge feature/user-system # 合并分支
git push origin main # 推送主分支
git branch -d feature/user-system # 删除本地分支

# 团队协作时, 推送新建分支, 并发起 PR
git push origin feature/user-system
```

```shell
git checkout main # 回到mian分支
git pull origin main # 本地更新main分支
git branch -d feature/user-system # 删除本地功能分支 -d为安全删除
```

```shell
git fetch --prune # 删除远程分支缓存, 若远程分支已删除, 则删除本地缓存
```


```shell
artix@Organ ~/P/7/7Artix.com (main)> git log --graph --oneline --all --decorate
* 3b042e4 (HEAD -> main, origin/main, origin/HEAD) feat: complete user management system
| * 31aad55 (origin/feature/user-system) feat: complete user management system
|/  
* 5f1d782 feat: implement core website framework and base features
* 5391003 Initial commit
* b98607d Initial commit
artix@Organ ~/P/7/7Artix.com (main)> git fetch --prune
From https://github.com/7Artix/7Artix.com
 - [deleted]         (none)     -> origin/feature/user-system
artix@Organ ~/P/7/7Artix.com (main)> git log --graph --oneline --all --decorate
* 3b042e4 (HEAD -> main, origin/main, origin/HEAD) feat: complete user management system
* 5f1d782 feat: implement core website framework and base features
* 5391003 Initial commit
* b98607d Initial commit
```

添加 github ssh 认证连接

```shell
ssh-keygen -t ed25519 -C "witch_key"
cat ~/.ssh/id_ed25519.pub
```

在 github 添加新 SSH 授权, 然后在服务器测试:

```shell
ssh -T git@github.com
```

