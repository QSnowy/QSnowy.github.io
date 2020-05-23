---
title: Mac下Cocoapods的安装和使用
tags: [Mac, 开发环境, cocoapods]
---



Cocoapods 是Objective-C工程项目中，常用的第三方库管理工具，可以方便的升级和管理项目中依赖的第三方库。

<!-- more -->

#### 1. Cocoapods 环境的安装

 Cocoapods工具是建立的Ruby基础上的，由于Mac自带Ruby，所以就不用我们自己安装Ruby环境了，但是由于众所周知的原因，国内还是要替换一下ruby源。

移除官方源

```shell
gem sources --remove https://rubygems.org/
```

添加国内源

```shell
gem source -a https://gems.ruby-china.com
```

看一下当前是否成功替换ruby源

```shell
gem sources -l
// https://gems.ruby-china.com/
```

开始安装

```shell
sudo gem install cocoapods
// OSX 10.11后，由于部分系统文件夹权限问题，所以使用下面的安装命令
sudo gem install -n /usr/local/bin cocoapods
```

检测是否安装成功

```shell
pod search AFNet
```

如果有搜索结果出来，说明已经安装成功了。

#### 2. Podfile文件的创建

在工程目录下，执行 `pod install` 时，`pod` 会解析 `Podfile` 文件，检测依赖，拉取相关三方库，最终生成 `.xcworkspace` 文件。文件格式大致如下

```ruby
platform : ios, '7.0'
#use_frameworks!

target 'your project target name' do
pod 'AFNetworking'
end
```

打开工程目录，在终端中创建 `Podfile`

```shell
vi Podfile
```

> vim的一些操作命令 
> `i` 进入编辑模式 开始编辑podfile文件  `esc` 退出编辑  `:wq` 保存并退出

#### 3. pod spec索引仓库下载太慢怎么办

由于pod spec索引库都是存放在的GitHub上面的，索引库中文件很多很大，经常出现无法安装的问题。虽然Cocoapods在1.8.0加入了CDN，加快下载速度，在网络环境较差的情况下，依旧难搞。

**解决方案：替换国内的镜像库**

移除现有的master索引库

```shell
rm -rf ~/.cocoapods/repos/master
// 或者
pod repo remove master
```

添加国内镜像

```shell
// 清华镜像库
git clone https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git ~/.cocoapods/repos/master
// 码云镜像库
git clone https://gitee.com/mirrors/CocoaPods-Specs.git ~/.cocoapods/repos/master
```

#### 4.Cocoapods的常用命令

查看pod版本

```shell
pod --version
```

更新gem

```shell
sudo gem update --system
```

更新pod，就是再次安装cocoapods

```shell
sudo gem install -n /usr/local/bin cocoapods
```

