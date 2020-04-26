---
title: 使用hexo搭建GithubPage博客
date: 2020-04-26 08:19:50
tags:

---

[Hexo](https://hexo.io/zh-cn/)是一个基于[Node.js](https://nodejs.org/)的高效静态网站生成框架，使你可以专注于MarkDown文档内容的编写，其他的网站资源生成交给Hexo就行了。我主要讲一讲自己是如何在Mac上面使用Hexo编写博客的。

### 创建博客

首先你得安装[Node.js](https://nodejs.org/)和[Git](https://git-scm.com/)，不过Mac里面默认自带Git的，不用重复安装了。Node环境，这里我用的是[Homebrew](https://brew.sh/index_zh-cn)安装的

直接终端里输入

```shell
brew install node
```

安装Hexo

```shell
npm install hexo
```

本地创建博客目录

```shell
hexo init <blog directory>
```

进入博客目录，运行启动服务器命令

```shell
hexo serve
// 也可以简写为 hexo s
```

之后终端会输出以下信息

>  INFO  Start processing
>  INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.

表明博客已经运行在[本地](http://localhost:4000)了，浏览器打开网址就能看到效果，是不是很简单。

### 部署博客到GithubPages

下面我们就要把自己的博客部署到[GitHubPages](https://github.com)，首先你得有个GitHub交友账号，这里默认各位已经有了。我们去GitHub上面创建一个repository，名字要这个样式的：`<yourname>.github.io`，因为GitHubPages会默认读取这个名字的仓库。

安装部署git的插件

```shell
npm install hexo-deployer-git --save
```

设置一下站点配置文件`_config.yml`

```javascript
deploy:
  type: git
  repository: 'your repository address' # 自己的GitHub仓库地址
  branch: master # 指定一下 要放部署博客的分支
```

一键部署

```shell
hexo deploy
// 可以简写为 hexo d
```

等几分钟，去浏览器打开 `<yourname>.github.io` 就能看到自己的博客了。



### 利用Travis自动部署

还想更简单吗，只用push文章到GitHub，然后自动部署到GitHubPages？

