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

新建  `Hello World` 程序，并编译该文件

```c
#include <stdio.h>
int main(int argc, char ** argv) {
	printf("Hello World"); 
}
```

将c语言文件，编译为html文件

```shell
emcc hello.c -s WASM=1 -o hello.html
```

创建一个本地 `web server`，展示编译后文件

```shell
emrun --no_browser --port 8080 .
```

打开改[本地网站](http://localhost:8080/hello.html)，预览效果，可以在Emscripten控制台中看到  `Hello World` 输出信息。

