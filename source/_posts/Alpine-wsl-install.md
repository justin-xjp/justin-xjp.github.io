---
title: hexo环境的搭建
date: '2019-11-13 13:19'
tags:
  - linux
  - Alpine
  - nodejs
  - npm
  - hexo
toc: true
categories:
  - 瞎折腾
abbrlink: a0c259b2
---
年纪大了，时间一久就容易忘掉很多事情，本文记录了在win10的wsl上配置hexo环境的过程，希望哪天重建的时候还能适用。

为什么在wsl上安装：

- 有全套的软件环境
- 删掉wsl可以将整个环境清理
- 工作离不开windows

<!--more-->

# Alpine 的安装

## 谁是Alpine

> Alpine Linux is a security-oriented, lightweight Linux distribution based on musl libc and busybox.
>
> Alpine Linux 是一个面向安全，轻量级的基于musl libc与busybox项目的Linux发行版。

是在docker中很常用的一种linux，wsl安装后只占用8M左右的空间，运行起来更是节省内存，就像是开着一个vbox的lede。

## 安装

首先是打开WSL的许可，有个powershell的命令但是我没记住。可以从`控制面板`>`程序`>`开启关闭windows功能`的地方找到。

进入win store安装Alpine。

## 更改用户配置

以下内容来自[jlelse's Blog](https://jlelse.blog/dev/using-windows-3/):

> By default you start the Alpine WSL with an unprivileged user. I setup the ability to use sudo, by following these steps (*I hope I remembered them all correctly*):
>
> 1. logging in with root (`wsl.exe --distribution Alpine --user root`),
> 2. setting a root password (`passwd`),
> 3. installing sudo (`apk add sudo`),
> 4. un-commenting a line in the `/etc/sudoers` file to allow anyone use sudo who is in the sudo group (`%sudo ALL=(ALL) ALL`),
> 5. create a sudo group (`addgroup sudo`) and
> 6. add my default user to that group (`usermod -aG sudo <username>`).
> 7. After that set the password for the default user (`passwd <username>`), close and reopen the Alpine console.

这样就可以适用sudo来操作了。

接下来是更换国内镜像：

```bash
sudo sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories 
```

OK。一个超mini的linux安好了，相对于流行的ubuntu for wsl那动辄上G的身材，Alphi的几十M简直可以忽略。

其余的信息，建议多看看[wiki](https://wiki.alpinelinux.org/wiki/Main_Page)

# 安装nodejs 和npm

### apk 安装

```
$sudo apk add npm
(1/8) Installing ca-certificates (20190108-r0)
(2/8) Installing c-ares (1.15.0-r0)
(3/8) Installing libgcc (8.3.0-r0)
(4/8) Installing http-parser (2.9.2-r0)
(5/8) Installing libstdc++ (8.3.0-r0)
(6/8) Installing libuv (1.29.1-r0)
(7/8) Installing nodejs (10.16.3-r0)
(8/8) Installing npm (10.16.3-r0)
Executing busybox-1.30.1-r2.trigger
Executing ca-certificates-20190108-r0.trigger ██████████████████████████████████
OK: 63 MiB in 25 packages
```

```
node -v
npm -v
```



## 权限问题引起的EACCES错误

`npm install -g`全局安装时会引起**EACCES**。最简单粗暴的解决办法是sudo安装，但会造成权限的滥用。

解决办法来自[Resolving EACCES permissions errors when installing packages globally](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally#manually-change-npms-default-directory)

> To minimize the chance of permissions errors, you can configure npm to use a different directory. In this example, you will create and use hidden directory in your home directory.
>
> 
>
> 1. Back up your computer.
>
> 2. On the command line, in your home directory, create a directory for global installations:    
>
> ```
> mkdir ~/.npm-global
> ```
>
> 3. Configure npm to use the new directory path:    
>
> ```
> npm config set prefix '~/.npm-global'
> ```
>
> 4. In your preferred text editor, open or create a 
>
> ```
> ~/.profile
> ```
>
> file and add this line:    
>
> ```
> export PATH=~/.npm-global/bin:$PATH
> ```
>
> 5. On the command line, update your system variables:    
>
> ```
> source ~/.profile
> ```
>
> 6. To test your new configuration, install a package globally without using `sudo`
>
> ```
> npm install -g cnpm
> ```
>
> 
>
>
> Instead of steps 2-4, you can use the corresponding ENV variable (e.g. if you don’t want to modify `~/.profile`):
>
> 
>
> ```
> NPM_CONFIG_PREFIX=~/.npm-global
> ```

## 换淘宝源

```
sudo npm install -g cnpm
```

# hexo的安装


```bash
cnpm install hexo-cli -g
hexo init blog #会执行git clone，没有git的会报错
cd blog
cnpm install
hexo server
```

**PS:** Alpine wsl的git安装完后会比较占空间，似乎是因为硬连接造成的，`git`的安装目录下有很多一样大小的文件，基于以前的经验，这些文件本应该是硬连接，但因为存储在win下就占用了更多的空间，查看过ubuntu-wsl，似乎可以用`ln -s`软连接代替，应该不会有大的影响。关于win10下的硬连接，以前好像看到过能够解决。

**hexo**的配置，官方文档[^1]很详细。

主题选择了**maupassan** [^2]

书写md文件，一直习惯使用Typora，`hexo-renderer-markdown`完整的实现了Typora的语法，其实`hexo-grenderer-markdown-it`应该也能才对，但我一直没有配置成功。不知道为什么……

## 更新本地hexo

- `cnpm i hexo-cli -g`
- `cnpm install -g cnpm-check`
- `npm-check`
- `npm-check -u`更新，并--save在了`package.json`里 , #只做到了这里
- `cnpm install  -g npm-upgrade`
- `npm-upgrade`
- `npm update -g`
- `npm update --save`

## 最近更换了主题

升级了本地环境以后，maupassan主题出现了问题。只好随手换成了目前最流行的**next**，更换起来很简单，配置也很容易：

```
cnpm i hexo-theme-next
```
更改`_config.yml`设置
```
theme: next
```
注意空格

复制出**next**的配置文件
```
cp node_modules/hexo-theme-next/_config.yml _config.next.yml
```
增加 **标签**和**分类**页面
```
hexo new page tags
hexo new page categories
```
分别在文件中增设**type**

```
---
title: about
date: 2019-06-25 19:16:17
type: "about"
---
 
---
title: tags
date: 2019-06-25 19:16:17
type: "tags"
---
 
---
title: categories
date: 2019-06-25 19:16:17
type: "categories"
---
```
修订设置文件`_config.next.yml`

```
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```

页面动画更改`_config.next.yml`

```
motion:
  enable: true
  async: false
  transition:
    # All available transition variants: https://theme-next.js.org/animate/
    # Transition variants:
    # fadeIn | fadeOut | flipXIn | flipXOut | flipYIn | flipYOut | flipBounceXIn | flipBounceXOut | flipBounceYIn | flipBounceYOut
    # swoopIn | swoopOut | whirlIn | whirlOut | shrinkIn | shrinkOut | expandIn | expandOut
    # bounceIn | bounceOut | bounceUpIn | bounceUpOut | bounceDownIn | bounceDownOut | bounceLeftIn | bounceLeftOut | bounceRightIn | bounceRightOut
    # slideUpIn | slideUpOut | slideDownIn | slideDownOut | slideLeftIn | slideLeftOut | slideRightIn | slideRightOut
    # slideUpBigIn | slideUpBigOut | slideDownBigIn | slideDownBigOut | slideLeftBigIn | slideLeftBigOut | slideRightBigIn | slideRightBigOut
    # perspectiveUpIn | perspectiveUpOut | perspectiveDownIn | perspectiveDownOut | perspectiveLeftIn | perspectiveLeftOut | perspectiveRightIn | perspectiveRightOut
    post_block: fadeIn
    post_header: slideDownIn
    post_body: slideDownIn
    coll_header: slideLeftIn
    # Only for Pisces | Gemini.
    sidebar: slideUpIn

    post_block: fadeIn
    post_header: fadeInDown
    post_body: fadeInDown
    coll_header: fadeInLeft
    # Only for Pisces | Gemini.
    sidebar: fadeInUp
```

更改`Next`主题样式`_config.next.yml`：

```
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```



注意空格。
整体来说，我仍然觉得next的动画有些花哨，我更喜欢以前的简洁，所以`false`掉了。但这么简单的设置是我非常满意的地方。

---

[^1]: https://hexo.io/zh-cn/docs
[^2]: https://www.haomwei.com/technology/maupassant-hexo.html