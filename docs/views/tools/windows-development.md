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
个人在 Windows 下的工具清单和开发环境。
:::

<!-- more -->

## 工具清单

- 浏览器：Edge、Chrome
- 编辑器：VS Code、Typora
- 终端：Windows Terminal Preview、oh my zsh(WSL2)
- 效率工具：Everything、Wox、Quicker、Ditto
- 待办清单：Microsoft To Do
- 压缩：7-Zip
- 词典：欧路词典
- 思维导图：XMind
- 截图：Snipaste
- 录屏：ScreenToGif
- 图床：PicGo + Github
- 播放器：PotPlayer
- 其他：Shadowsocks、F.lux、MacType

## WSL 2 环境搭建

### 安装 WSL 2

- 安装 wsl

  ```bash
  # 管理员权限
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  ```

- 重启系统
- 启用 `Virtual Machine Platform`

  ```bash
  # 管理员权限
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```

- 重启系统
- 下载并安装 WSL 2 Linux 内核：[wsl_update_x64](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)
- 设置 WSL 2 为默认版本

  ```bash
  wsl --set-default-version 2
  ```

- 打开[Microsoft 应用商店](https://aka.ms/wslstore)下载相应的 linux 发行版,
- 安装完成后运行，设置用户名/密码即可
- 以下都以 `Ubuntu-20.04` 为例

### 安装 oh my zsh

- 安装 zsh

  ```bash
  sudo apt update
  sudo apt install zsh
  chsh -s /bin/zsh
  # 重启 ubuntu
  touch ~/.zshrc
  ```

- 安装 oh my zsh

  ```bash
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

  # 自动提示插件
  git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

  # 高亮插件
  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

  vim ~/.zshrc

  # 添加插件
  plugins=(git zsh-autosuggestions zsh-syntax-highlighting)

  source ~/.zshrc
  ```

### Terminal 默认使用 WSL

- 打开 Terminal
- 点击 **设置**
- 将 `defaultProfile` 改成 WSL 的 `guid` 即可

## node 环境

- 安装 nvm

  ```bash
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash

  vim ~/.zshrc

  # 添加环境变量
  export NVM_DIR="$HOME/.nvm"
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
  [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

  source ~/.zshrc
  ```

- 安装 node

  ```bash
  nvm install 12
  ```

- 安装 nrm

  ```bash
  npm install -g nrm
  nrm test
  nrm use taobao
  ```

## Git 配置

```bash
git config --global user.name danranvm
git config --global user.email danranvm@gmail.com

ssh-keygen -t ed25519 -C "danranvm@gmail.com"
#  ssh-keygen -t rsa -b 4096 -C "danranvm@gmail.com"
cat ~/.ssh/id_rsa.pub
```

## VS Code 插件

- LOCAL
  - Bracket Pair Colorizer 2
  - Debugger for Chrome
  - Remote - WSL
  - Remote - SSH
  - vscode-icons
- WSL:UBUNTU
  - Code Spell Checker
  - GitLens
  - Markdown Preview Enhanced
  - Prettier - Code formatter
  - Todo Tree

Tips: VS Code 已经内置了 `Setting Sync`

## 完成

至此，Windows 开发环境基本上搭建完毕，但愿 WSL2 不要让我失望，让我少踩点坑。
