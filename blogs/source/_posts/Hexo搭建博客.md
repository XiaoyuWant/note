---
title: Hexo搭建博客
date: 2022-06-20
---

# 使用 Hexo + Github & Github pages 搭建博客

新建一个仓库，main 分支存储 hexo 相关内容，生成的静态页面在 gh-pages 分支，看起来是一个比较好的解决方案。

## 1. 准备工作 
1. 在github上新建一个 repository，比如叫 `note`
2. 新增 `main` 和 `gh-pages` 分支
3. 在仓库的设置项找到 page，设置为`gh-pages`分支
4. 在本地克隆这个 repository
5. 安装 `nodejs` &  `hexo` & `hexo-deployer-git`

## 2. 开始操作
1. 在仓库主目录新建hexo项目 `hexo init blog`
2. 进入 `blog` 目录，安装需要的模块`npm install`
3. 下载主题：进入themes文件夹下载主题如next
4. 修改主目录的 `_config.yaml`:
    ```
    # 修改主页路径：
    url: xiaoyuwant.github.io/note
    root: /note/
    ...
    # 新增部署方案：
    deploy:
        type: git
        repository: git@github.com:XiaoyuWant/note.git
        branch: gh-pages

    ```
## 3. 写作 
- 在仓库的主目录下 `./source/_post/` 文件夹存储markdown文件
- 文件头，如：
    ```
    ---
    title: VsCode SSH to Server
    date: 2021-04-15
    ---
    正文

    ```
## 4. 部署方法
- 在hexo目录，执行 `hexo g` 生成静态文件
- 本地测试效果：`hexo s`
- 部署到github：`hexo g -d`

