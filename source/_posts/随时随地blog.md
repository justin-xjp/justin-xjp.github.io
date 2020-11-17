---
title: 随时随地blog
toc: true
mathjax: true
abbrlink: '99893367'
tags:
  - hexo
date: 2020-11-16 14:29:07
categories:
  - 瞎折腾
---



 假设在一台新的机子上，如何继续编写更新hexo呢？

# 一、配置编写环境

参见[hexo环境搭设](a0c259b2)，配置 git, nodejs, npm.

- 使用git clone ，得到blog 的编译目录 ./blog
- 进入blog目录，并使用`npm install `完成hexo环境的搭建。

## 升级本地环境

对于本地环境的升级操作，以下内容来自于网络，并很有可能已经过时了。

<!--more-->

- `cnpm i hexo-cli -g`
- `cnpm install -g cnpm-check`
- `npm-check`
- `npm-check -u`更新，并--save在了`package.json`里
- `cnpm install  -g npm-upgrade`
- `npm-upgrade`
- `npm update -g`
- `npm update --save`

因为安装一个插件，遵从`npm audit`给出的报告使用`npm i hexo@5.2.0`更新了单独的文件。

# 二、 开始编写blog

- `hexo new [layout] <title>`新建文章
- 文章中使用`[连接名称](abbrlink_num)`建立文章间的链接。

## hexo 中的一些使用技巧

### 1. md中的图片

hexo中默认图片的引用方法与本地的markdown方法不一致。

在Typora中，图片根据我的设置是从文件同名附件目录`./${filename}`中调用的：
```
![注释文字](图片文件名.jpg)
```

而在hexo中，可以修改配置文件`_config.yml`，根据[官方文档](https://hexo.io/zh-cn/docs)设置`post_asset_floder:true`，这样会从同名目录中引用图片，和Trpora 的图片设置类似。同时，注意修改文件引用语法:

```
{% asset_img xxxx.jpg "图片的文字描述" %}
```

引起的问题就是在typora中不再能明确的得到预览图片。

#### 解决

同时使用hexo 和 Typora 的人还是很多的，有篇文章[^1]声称解决了此问题，试试看：

```
npm install hexo-image-link --save
```

然后可以使用默认的markdown语法本地显示图片。用`hexo server -debug`验证下。

![挑个图片看看](随时随地blog/2.png)

`! [挑个图片看看](随时随地blog/2.png)`

成功了，但引入了另一个问题，增加的插件似乎判断"`![xxx](xxx)`"格式并直接替代，但没有判断此代码是不是在代码块中，造成代码块中的演示也被替换成了图片。于是我只能在代码块的叹号！后面加个空格。

### 2. 简要标记

可以使用`<!--more-->`标记，使得文章在主页中简略显示。

### 3.调试、生成

```
$ hexo server
```

hexo会在`http://localhost:4000/`生成服务，并监测文件变动自动更新。也就是说可以在编写过程中时时在本地看到生成后的效果。

```
$ hexo generate
```

这个命令会在`public`目录下生成静态文件，可以将此文件下的文件发布到需要的地方。

### 4.自动部署

```
$ hexo generate --deploy
$ hexo deploy --generate
```

以上两个命令都可以生成并部署生成的静态页面到github , 要实现自动部署，需要做以下设置：

- 修改`_config.yml`中的参数

  ```
  deploy:
  	type: git
  ```

- 安装`hexo-deploer-git`

  ```
  $ npm install hexo-deployer-git --save
  ```

  并修改配置为：

  ```
  deploy:
    type: git
    repo: <repository url> 
    branch: gh-pages
    message: 
  ```

  

简写：

```
$ hexo g -d
$ hexo d -g
```



# 三、git同步

- `git st`
- `git add ...`
- `git ci -m "commit message"`
- `git push <origin> <branches>`
- git命令的简写来自“廖雪峰的git教程”



直接`git push origin dev`的话，会提示

> Warning: Permanently added the RSA host key for IP address xxx...

大概是说SSH连接缺失公钥，需要将目前计算机的公钥添加到github网站。然后就可以顺利更新。





---

[^1]:https://www.cnblogs.com/cocowool/p/hexo-image-link.html