---
title: 使用Hexo搭建GithubPage博客
date: 2020-04-26 08:19:50
categories: Mac
tags: [Hexo, NodeJS]

---

[Hexo](https://hexo.io/zh-cn/)是一个基于[Node.js](https://nodejs.org/)的高效静态网站生成框架，使你可以专注于MarkDown文档内容的编写，其他的网站资源生成交给Hexo就行了。我主要讲一讲自己是如何在Mac上面使用Hexo编写博客的。
<!-- more -->

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

```yaml
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

还想更简单吗，只用push文章到GitHub，然后自动部署到GitHubPages？利用 [Travis](https://travis-ci.org/) 就可以实现。

使用GitHub账号登陆Travis， 在 [GitHubApplications](https://github.com/settings/applications) 能看到被授权的Travis应用。

然后在 [GithubInstallActions](https://github.com/settings/installations) 配置一下Travis可以访问的博客仓库。

然后在 [GitHubSetting](https://github.com/settings/tokens) 生成一个token，将这个token复制下来，回到 [Travis](https://travis-ci.com/getting_started) 中，在仓库 `<yourname>.github.io` 配置一下`Environment Variables`，添加下面一行键值对

![ghtoken](https://snowyblog.oss-cn-shenzhen.aliyuncs.com/Travis_GH_TOKEN.png)

在博客目录下，新建一个自动编译配置文件`.travis.yml`，注意目前GitHubPages的默认的站点分支是master，无法指定gh-pages了。

```yaml
# 指定语言环境
language: node_js
# 指定是否需要sudo权限
sudo: false
# 指定node_js版本
node_js:
  - 10.0
# 指定缓存模块，可选，加快编译速度
cache: npm
# 指定博客源码分支
branches:
  only:
    - hexo
# 清除之前缓存，生成新网站资源
script:
  - hexo clean
  - hexo generate
# 部署
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    all_branches: true # solve a permission problem
  target_branch: master
  local-dir: public
```

### Hexo常用配置

文章多标签的设置
```markdown
tags: [tag1, tag2, ...]
```
关闭某个文章的评论功能，前提是开启了评论插件

```markdown
comments: false
```

### next主题配置

主题配置文件路径：`themes/next/_config.yml`

#### 开启评论功能

注册一个 [Disqus](https://disqus.com/) 账号，在next主题目录的_config.yml修改如下配置
```yaml
disqus:
  enable: true
  shortname: your disqus short name
  count: false # 是否展示文章评论数
```

#### 添加标签页面
1. 修改主题配置：

```yaml
menu:
  tags: /tags/ || fa fa-tags
```

2. Hexo创建tags文件夹

```shell
hexo new page tags
```
3. 修改tags/index.md文件

```yaml
type: "tag"
comments: false
```
