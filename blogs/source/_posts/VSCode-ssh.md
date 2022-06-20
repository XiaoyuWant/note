---
title: VsCode SSH to Server
date: 2021-04-15
---
# VsCode SSH to Server

## How To Use Vscode To Edit Your Code On Sever?

### 1. On Windows System

- 在 windows 开始中搜索 `服务` ，找到 `OpenSSH` 并启动
- 打开 CMD，使用 `ssh-keygen` 命令生成密钥，在 `C:\Users\'YOUR_NAME'\.ssh\id_rsa.pub`
- 编辑 `C:\Users\'YOUR_NAME'\.ssh\config` 的内容：
    
    ```
    Host ANY_NAME
      HostName YOUR_SEVER_HOST
      User YOUR_USER_NAME
      Port YOUR_PORT
    ```
    
    其中端口默认为22，可不写 `Port` 行；若为其他端口，请指明。
    

### 2. On Server

- 复制上一步 `C:\Users\'YOUR_NAME'\.ssh\id_rsa.pub` 的内容并添加到到服务器文件 `home/.ssh/authorized_keys` 中

### 3. In Vscode

- 在Vscode插件市场下载 `remote-ssh`
- 按 `F1` 选择 `REMOTE_SSH: Connect to Host...`
- 选择之前设置好的 `ANY_NAME`
- 连接成功后，即可在 Vscode 左侧的 `explorer` 选项卡选择服务器文件路径

### 4. 大功告成！