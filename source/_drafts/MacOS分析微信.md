---
title: MacOS
---

[出自](https://bbs.pediy.com/thread-266041.htm)

### 准备工作：

- [Hopper Disassembler](https://www.hopperapp.com/index.html)
- [Class-dump](http://stevenygard.com/projects/class-dump/)
- [Frida](https://frida.re/)

一些小细节：将`class-dump` 的二进制文件放入 `/usr/local/bin` 或其他环境变量路径下，就能在任何路径下直接调用 `class-dump` 命令了

![](https://tva1.sinaimg.cn/large/008eGmZEgy1gnu36gj8z7j30ej03y3yy.jpg)

#### dump头文件

利用Class-dump拿到微信头文件，记得先建一个文件夹，在文件夹下执行

```shell
mkdir Wechat
cd Wechat
class-dump -H /Applications/WeChat.app
```

执行成功后，会生成很多头文件

#### 分析头文件

我们要找消息相关的文件，必定是Message*之类的命名了，最终定位到