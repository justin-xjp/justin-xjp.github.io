---
title: Alpine wsl 安装nvm和npm
date: '2019-11-14 12:48'
tags:
  - linux
  - Alpine
  - nodejs
  - npm
  - nvm
toc: true
categories:
  - 瞎折腾
abbrlink: 4d7ef6c2
---

一次失败的经历

<!--more-->

## nvm的安装

前期通过APK包管理安装了npm和其依赖，但会引起权限问题（EACCES）。查看了[hexo的文档](https://hexo.io/zh-cn/docs/#安装-Node-js)，其给出的解决办法有两种：

1. 使用nvm管理器安装node和npm以及hexo，不会引起sudo的过度使用
2. 手动配置全局安装位置及引用，避免权限不足。

决定尝试nvm的安装

### 卸载全局安装的node/npm

曾试过 apk del npm，结果连附带的依赖都自动卸载了，并且手动二进制安装并没有效果，（ubuntu 使用同样的二级制安装方法已经解决问题，可能跟没有删除依赖包有关。）以下内容来自网络[^1] :

```bash
#查看已经安装在全局的模块，以便删除这些全局模块后再按照不同的 node 版本重新进行全局安装
npm ls -g --depth=0 
#删除全局 node_modules 目录
sudo rm -rf /usr/local/lib/node_modules
#删除 node
sudo rm /usr/local/bin/node
#删除全局 node 模块注册的软链
cd /usr/local/bin && ls -l | grep "../lib/node_modules/" | awk '{print $9}'| xargs sudo rm
```


### 安装

遵循指南，使用脚本安装nvm[^2] [^3]

Alpine用的是busybox，只有ash。

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | ash
```

#### 关于源

在linux下，nvm读取的是环境变量`NVM_NODEJS_ORG_MIRROR`和`NVM_IOJS_ORG_MIRROR`。目前学习尚浅，还不知道 iojs是什么，但如果你想换源的话，可以在配置文件，`.profile`之类的中加入:

```
export NVM_NODEJS_ORG_MIRROR = "https://npm.taobao.org/mirrors/node"
```

### 另一个失败

上面的安装是在`apk install npm`后，完整的安装了依赖并且能够使用的前提下去手动删除`nodejs`和`npm`的，如果没有通过 apk 安装过npm，纯粹的安装nvm或是手动下载nodejs和npm的包来使用，都会在最终碰到问题：

即是手动安装过依赖的包也依然提示找不到node，虽然已经添加了环境变量，也能在ash中用`tab`键看到node命令，可就是不能执行。大概是哪里没搞懂，ubuntu下就没有问题

## 使用

### 常用命令
```
nvm install ## 安装指定版本，可模糊安装，如：安装v6.2.0，既可nvm install 6.2.0，又可nvm install 6.2
nvm uninstall ## 删除已安装的指定版本，语法与install类似
nvm use ## 切换使用指定的版本node
nvm ls ## 列出所有安装的版本
nvm ls-remote ## 列出所以远程服务器的版本（官方node version list）
nvm current ## 显示当前的版本
nvm alias ## 给不同的版本号添加别名
nvm unalias ## 删除已定义的别名
nvm reinstall-packages ## 在当前版本node环境下，重新全局安装指定版本号的npm包

```

#### 碰到了异常

```bash
$nvm install 12.13
Downloading and installing node v12.13.0...
Downloading https://nodejs.org/dist/v12.13.0/node-v12.13.0-linux-x64.tar.gz...
wget: unrecognized option: progress=bar
BusyBox v1.30.1 (2019-10-26 11:23:07 UTC) multi-call binary.

Usage: wget [-c|--continue] [--spider] [-q|--quiet] [-O|--output-document FILE]
        [--header 'header: value'] [-Y|--proxy on/off] [-P DIR]
        [-S|--server-response] [-U|--user-agent AGENT] [-T SEC] URL...

Retrieve files via HTTP or FTP

        --spider        Only check URL existence: $? is 0 if exists
        -c              Continue retrieval of aborted transfer
        -q              Quiet
        -P DIR          Save to DIR (default .)
        -S              Show server response
        -T SEC          Network read timeout is SEC seconds
        -O FILE         Save to FILE ('-' for stdout)
        -U STR          Use STR for User-Agent header
        -Y on/off       Use proxy
Binary download from https://nodejs.org/dist/v12.13.0/node-v12.13.0-linux-x64.tar.gz failed, trying source.
grep: /home/rocxer/.nvm/.cache/bin/node-v12.13.0-linux-x64/node-v12.13.0-linux-x64.tar.gz: No such file or directory
Provided file to checksum does not exist.
Binary download failed, trying source.
Downloading https://nodejs.org/dist/v12.13.0/node-v12.13.0.tar.gz...
wget: unrecognized option: progress=bar
BusyBox v1.30.1 (2019-10-26 11:23:07 UTC) multi-call binary.

Usage: wget [-c|--continue] [--spider] [-q|--quiet] [-O|--output-document FILE]
        [--header 'header: value'] [-Y|--proxy on/off] [-P DIR]
        [-S|--server-response] [-U|--user-agent AGENT] [-T SEC] URL...

Retrieve files via HTTP or FTP

        --spider        Only check URL existence: $? is 0 if exists
        -c              Continue retrieval of aborted transfer
        -q              Quiet
        -P DIR          Save to DIR (default .)
        -S              Show server response
        -T SEC          Network read timeout is SEC seconds
        -O FILE         Save to FILE ('-' for stdout)
        -U STR          Use STR for User-Agent header
        -Y on/off       Use proxy
Binary download from https://nodejs.org/dist/v12.13.0/node-v12.13.0.tar.gz failed, trying source.
grep: /home/rocxer/.nvm/.cache/src/node-v12.13.0/node-v12.13.0.tar.gz: No such file or directory
Provided file to checksum does not exist.
```

~~从反馈看，似乎是一个不同的参数造成了**wget** 没有那计划工作，对照源代码，我尝试修改了`nvm.sh`文件，希望能跳过这个错误。~~居然想解决提出问题的人，呵呵，简单说就是**Alpine**用的busybox，而busybox的**wget**实现和真正的**wget**有区别，下一个新的wget就好了

```
sudo apk add wget
```

再次安装

```bash
$nvm install 12.13
Downloading and installing node v12.13.0...
Downloading https://nodejs.org/dist/v12.13.0/node-v12.13.0-linux-x64.tar.gz...
--2019-11-14 09:19:33--  https://nodejs.org/dist/v12.13.0/node-v12.13.0-linux-x64.tar.gz
Resolving nodejs.org... 104.20.23.46, 104.20.22.46, 2606:4700:10::6814:172e, ...
Connecting to nodejs.org|104.20.23.46|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 21674628 (20M) [application/gzip]
Saving to: '/home/rocxer/.nvm/.cache/bin/node-v12.13.0-linux-x64/node-v12.13.0-linux-x64.tar.gz'

/bin/node-v12.13.0-linux-x64/  32%[==>               ]   6.70M   616KB/s    eta 22s
```

安装完成后

最终一直提示

```
$
nvm is not compatible with the npm config "prefix" option: currently set to "/home/rocxer/npm"
Run `nvm use --delete-prefix v12.13.0` to unset it.
$nvm use --delete-prefix v12.13.0
Now using node v12.13.0 (npm v6.12.0)
#虽然这样提示了，但仍不能使用
```



---
## 参考

[^1]:  https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally
[^2]:  https://docs.npmjs.com/downloading-and-installing-node-js-and-npm 
[^3]: https://github.com/nvm-sh/nvm#install--update-script