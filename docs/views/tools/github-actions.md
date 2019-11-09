---
title: Goodbye Travis, Hello Actions!
date: 2019-11-09
categories:
  - Tools
tags:
  - Github
  - Actions
---

::: tip
[Github Actions](https://github.com/features/actions) 是 Github 推出的持续集成服务。
它功能强大，使用简单，本文演示了如何自动化构建发布一个应用到 Github Pages 或者你的远程服务器。
:::

<!-- more -->

## 前言

说起持续集成服务，大家都知道 [Travis](https://travis-ci.org/), 我本来也是打算使用 Travis 来自动化部署[我的博客](https://blog.danran.site), 但是每次打开 Travis 都慢到爆炸，然后正好看到了 Actions 的介绍，于是就体验了一番。相对来说，它更加的快捷（无论是访问速度还是构建速度）也和 Github 结合得更好，使用更为方便。

## 注册

由于 Actions 还没有正式发布（据说 13 号发布），所以需要到 <https://github.com/features/actions/signup> 进行注册。注册成功后，就可以在你的仓库中看到 Actions。

![Actions Icon](https://cdn.jsdelivr.net/gh/danranvm/image-hosting/images/20191109173238.png)

## 使用

### 创建工作流

看到上述 Actions 图标后，点击进入创建工作流界面， 可以看到 Github 给我们提供了一些模板，我们随意点击其中一个 `Set up this workflow` , 就会在我们仓库的 `.github/workflows` 目录下创建一个 `xxx.yml` 文件, 内容如下（不同的模板有所需区别）：

```yml
name: CI # 工作流名称，任意自定义

on: [push] # 触发条件

jobs: # 具体的工作流
  build: # 任务名称，任意自定义
    runs-on: ubuntu-latest # 运行环境，还可以选择 windows-latest or macOS-latest

    steps: # 任务步骤
      - uses: actions/checkout@v1 # 使用已经定义好的 Action
      - name: Run a one-line script # 步骤名称
        run: echo Hello, world! # 执行命令
      - name: Run a multi-line script
        run: |
          echo Add other actions to build,
          echo test, and deploy your project.
```

然后我们把这个文件提交到仓库后，再次点击 Actions ，就可以看到我们刚刚创建的工作流就已经被执行过一次了（因为我们提交文件的时候，触发了 push 条件）。

![All workflows](https://cdn.jsdelivr.net/gh/danranvm/image-hosting/images/20191109180226.png)

点击右侧工作流的名称（上图中的 CI）,就可以看到工作流详细的执行信息。

![Workflow detail](https://cdn.jsdelivr.net/gh/danranvm/image-hosting/images/20191109180457.png)

好了，大概了解了这些信息，我们现在来正式部署我们的应用吧。

### Github Pages 部署

由于需要把构建后的内容上传到 Github 仓库，所以需要按照 [Creating a personal access token for the command line](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) 文档中创建一个 Token，然后把这个令牌保存在当前仓库的 `Settings/Secrets`,也就是 <https://github.com/用户名/仓库名/settings/secrets>中。

![Settings Secrets](https://cdn.jsdelivr.net/gh/danranvm/image-hosting/images/20191109181619.png)

然后按照上面的步骤创建一个 `workflow` , 当然你也可以在本地项目中创建，然后 push。

```yml
name: Build and Deploy
on:
  push:
    branches:
      - master
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@master

      - name: Build and Deploy
        uses: JamesIves/github-pages-deploy-action@releases/v2
        env:
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          BASE_BRANCH: master # The branch the action should deploy from.
          BRANCH: gh-pages # The branch the action should deploy to.
          FOLDER: dist # The folder the action should deploy.
          BUILD_SCRIPT: npm install && npm run build # The build script the action should run prior to deploying.
```

触发工作流的条件设置成了 `master` 分支的 `push` 。  
使用了别人写好的两个 action ：`actions/checkout` 拉取代码、 `JamesIves/github-pages-deploy-action` 构建以及上传代码。  
`env` 是 `action` 执行所需要的一些环境变量和设置。

### 远程服务器部署

根 Github Pages 不同的是，远程服务器部署不需要 Github Token, 而是需要通过 SSH 进行上传文件, 所以需要在 `Settings/Secrets` 中配置 `HOST` `USERNAME` `PASSWORD` 等参数。

![Settings Secrets](https://cdn.jsdelivr.net/gh/danranvm/image-hosting/images/20191109184914.png)

```yml
name: build-and-deploy
on:
  push:
    branches:
      - master
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@master
      - name: Setup node
        uses: actions/setup-node@master
        with:
          node-version: 12
      - name: install and build
        run: npm install && npm run build
      - name: copy file via ssh password
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          password: ${{ secrets.PASSWORD }}
          rm: 1 # 上传前先清空目录
          source: 'dist' # 构建结果目录
          target: 'nginx/html/blog' # 远程服务器 nginx 目录
          strip_components: 1 # 如果不配置该参数，上传后的目录结构就是 nginx/html/blog/dist
```

这里我们使用了 `actions/checkout`, `actions/setup-node` 和 `appleboy/scp-action` 三个 action 分别做了下载，构建和上传的操作。

## 相关资料

- [Action 官网文档](https://help.github.com/en/actions)
- [Action 官网仓库](https://github.com/marketplace?type=actions)
- [JamesIves/github-pages-deploy-action 更多参数](https://github.com/marketplace/actions/deploy-to-github-pages)
- [appleboy/scp-action 更多参数](https://github.com/marketplace/actions/scp-files)
