Mac上的包管理工具HomeBrew在国内安装起来有点太慢了。

官方安装

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

下载安装脚本

```shell
curl -o brew_install.sh https://raw.githubusercontent.com/Homebrew/install/master/install.sh
```

更改镜像源

```shell
vi brew_install.sh
// 替换清华镜像
BREW_REPO = "https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git".freeze
```

执行安装脚本

```shell
. brew_install.sh
```

