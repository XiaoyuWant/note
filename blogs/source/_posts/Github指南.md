---
title: Github 指南
date: 2021-06-15
---
# Github 指南

1. 生成公钥 `ssh-keygen`
2. 复制 `~/.ssh/id_rsa.pub` 的内容到github设置里的ssh keys
3. 测试是否能连接成功 `ssh -T git@github.com`
4. 克隆一个仓库`git clone git@github.com:XiaoyuWant/***.git`
5. 拉取远程更改 `git pull`
6. 确认更改文件`git add .`删除文件 `git rm filename`
7. 查看状态 `git status`
8. Commit `git commit -m "commit message"`
9. 提交到main分支 `git push -u origin main` / `git push`