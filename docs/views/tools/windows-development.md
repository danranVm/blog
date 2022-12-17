---
title: Windows 开发环境搭建（WSL）
date: 2020-08-15
categories:
  - Windows
tags:
  - Windows
  - WSL
  - Development
---

::: tip
个人在 Windows 下的工具清单和开发环境，部分安装命令需要在科学上网环境下使用。
:::

<!-- more -->

## 工具清单

- 浏览器：Edge、Chrome
- 编辑器：VS Code、Typora
- 终端：Windows Terminal Preview
- IM: 微信、钉钉、口袋助理
- Office: WPS
- 思维导图：XMind
- 待办清单：Microsoft To Do
- 词典：有道词典
- 压缩：7-Zip
- 粘贴板：Ditto
- 截图：Snipaste
- 录屏：ScreenToGif
- 图床：PicGo
- 播放器：PotPlayer
- 远程：ToDesk
- 其他：Clash for Windows、SwitchHosts

## WSL 2 环境搭建

### 安装 WSL 2

- 安装 wsl

  - 一键式命令

    ```bash
    # 管理员权限,
    wsl --install
    # 如果遇到 Ubuntu 无法下载, 就结束进程，手动安装 Ubuntu
    ```
  - 重启系统

- 安装 Ubuntu (第一步没有正常安装 Ubuntu 的情况下)
  - 打开[Microsoft 应用商店](https://aka.ms/wslstore)下载相应的 linux 发行版
  - 安装完成后运行，设置用户名/密码即可

### oh my zsh

- 安装 zsh

  ```bash
  sudo apt update
  sudo apt install zsh
  chsh -s /bin/zsh
  # 重启 ubuntu, 根据提示输入：0
  ```

- 安装 oh my zsh

  ```bash
  # 如果遇到网络原因无法直接通过 curl 安装，可以手动安装：
  # 1. 下载 install.sh 文件，放入 wsl 中: //wsl$/Ubuntu/home/vm 
  # 2. 执行 sh install.sh
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

  # 自动提示插件
  git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

  # 高亮插件
  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

  vim ~/.zshrc

  # 添加插件
  plugins=(git z zsh-autosuggestions zsh-syntax-highlighting)

  source ~/.zshrc
  ```

### Terminal 默认使用 WSL

- 打开 Terminal
- 点击 **设置**
- 将 `defaultProfile` 改成 WSL 的 `guid` 
- 设置默认打开路径为 WSL 路径：`"startingDirectory": "//wsl$/Ubuntu/home/vm"`

## node 环境

- 安装 nvm

  ```bash
  # 如果遇到网络原因无法直接通过 curl 安装，可以手动安装, 参考 oh my zsh
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash

  source ~/.zshrc
  ```

- 安装 node

  ```bash
  # 查看所有版本
  nvm ls-remote

  # 安装指定版本
  nvm install 16

  # 或者也可以选择安装最后一个版本
  # nvm install node
  ```

- 安装 pnpm

  ```bash
  curl -fsSL https://get.pnpm.io/install.sh | sh -
  
  source ~/.zshrc
  ```

- 安装 nrm

  ```bash
  npm install -g nrm

  nrm test

  nrm use taobao
  ```

- 安装 http-server

  ```bash
  npm install -g http-server
  ```

## Git 配置

```bash
git config --global user.name danranvm
git config --global user.email danranvm@gmail.com

ssh-keygen -t ed25519 -C "danranvm@gmail.com"
cat ~/.ssh/id_ed25519.pub
```

## VS Code 插件

- LOCAL
  - Remote - WSL
  - Remote - SSH
  - vscode-icons
  - Markdown Preview Enhanced / Slidev
- WSL:UBUNTU
  - Angular Language Service
  - Code Spell Checker
  - ESLint
  - GitHub Copilot
  - GitLens
  - Prettier
  - Todo Tree
  - Volar

Tips: VS Code 已经内置了 `Setting Sync`
