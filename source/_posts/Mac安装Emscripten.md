---
layout: post
title: Mac安装Emscripten
date: 2020-12-18
categories: Mac
tags: [经验之谈]
---

按照[官网](https://emscripten.org/docs/getting_started/downloads.html])介绍进行环境配置，拉取代码到本地

```shell
# Get the emsdk repo
git clone https://github.com/emscripten-core/emsdk.git]
# Enter that directory
cd emsdk
```

下载并安装最新工具包

```shell
# Download and install the latest SDK tools.
./emsdk install latest
# Or use Python3 install and download tools.
python3 ./emsdk.py install latest

# Make the "latest" SDK "active" for the current user. (writes .emscripten file)
./emsdk activate latest

# Activate PATH and other environment variables in the current terminal
source ./emsdk_env.sh
```

但是当执行到  `./emsdk install latest` 时报错，`SSL error`，原因是 `emsdk.py` 文件有问题，在该文件加入如下代码，再次执行。

```python
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
```



