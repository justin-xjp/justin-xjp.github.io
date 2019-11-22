---
layout: timeline
title: 手动配置一个markdown-it环境
tags:
  - hexo
categories:
  - 瞎折腾
toc: true
mathjax: true
abbrlink: 1e400e37
date: 2019-11-22 14:20:10

---


误打误撞安装了`hexo-renderer-markdown`（下文中将简称`markdown插件`），发现我没有配置正确的`hexo-renderer-markdown-it`正常工作了，但此插件有些我不喜欢的元素，本着折腾的原则，打算学习中手动安装插件。

## 学习

学他人之长。

**markdown插件**[^1] 中包含了很少的文件:

- index.js
- package.json
- package-lock.json

包含了需要安装的包及依赖信息，是一个简单的npm包装。

其中主要引用了几个包：

```javascript
const def_pugs_list = [
    'markdown-it-abbr',
    'markdown-it-anchor',
    'markdown-it-attrs',
    'markdown-it-checkbox',
    'markdown-it-deflist',
    'markdown-it-emoji',
    'markdown-it-footnote',
    'markdown-it-ins',
    'markdown-it-katex',
    'markdown-it-mark',
    'markdown-it-sub',
    'markdown-it-sup'
];
```

## 安装环境

### 搭建一个新的blog

`hexo init newblog`

- 安装喜欢的主题

```
$git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
$ cnpm install hexo-renderer-pug --save
$ cnpm install hexo-renderer-sass --save
```

编辑Hexo目录下的 `_config.yml`，将`theme`的值改为`maupassant`，配置其他参数至满足要求。

- 添加测试用文件

使用**Typora**的markdown特性文件进行测试。在默认的情况下，大部分特性都没有展开，但默认的marked使用mathjax，渲染公式很漂亮。

{% asset_img 1574384667011.png %}

{% asset_img 1574384893999.png %}

因为解释器的冲突，`\\`被解释成了单斜杠, 可在线的mathjax的字体很漂亮很舒服。我一直担心katex的字体怎么才能折腾好。

还有markdown-it[官网](https://markdown-it.github.io/)的文本复制下来一并作为特性检验.

### 更换解释器

```
cnpm un hexo-renderer-marked --save

cnpm i hexo-renderer-markdown-it --save
npm i hexo-renderer-markdown-it --save
```

我遇到过几次安装完后运行`hexo server`提示：

```
FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
TypeError: opt.plugins.reduce is not a function
)
...........

```

原因找到了,因为用的配置文件里出现了不该出现的东西. 



默认没有更改`_config.yml`时, 直接渲染结果居然可以出现`mathjax`。那我只需要慢慢添加所需的其他插件就可以了。

更改`config.yml`[^2]，添加必要的配置：

```
markdown:
 render:
   html: true
   xhtmlOut: false
   breaks: true
   linkify: true
   typographer: true
   quotes: “”‘’
 anchors:
   level: 2
   collisionSuffix: 'v'
   permalink: false
   permalinkClass: header-anchor
   permalinkSymbol: ¶
```

新旧版本的差异：

在github页面，停更4年的`hexo-renderer-markdown-it`有更新，插件方面添加了更多的文件：

```
 "dependencies": {
-    "lodash.assign": "^3.2.0",
-    "markdown-it": "^5.0.1",
+    "markdown-it": "^10.0.0",
-    "markdown-it-abbr": "^1.0.0",
+    "markdown-it-abbr": "^1.0.4",
+    "markdown-it-cjk-breaks": "^1.1.2",
+    "markdown-it-container": "^2.0.0",
+    "markdown-it-deflist": "^2.0.3",
+    "markdown-it-emoji": "^1.4.0",
-    "markdown-it-footnote": "^2.0.0",
+    "markdown-it-footnote": "^3.0.1",
-    "markdown-it-ins": "^2.0.0",
+    "markdown-it-ins": "^3.0.0",
+    "markdown-it-mark": "^3.0.0",
    "markdown-it-sub": "^1.0.0",
    "markdown-it-sup": "^1.0.0",
    "sluggo": "^0.2.0"

  },
```



### 更换解释器二

决定安装git库的新版本hexo-markdown-it

```
npm un hexo-renderer-markdown-it --save
```

直接安装库里的包[^3]:

```
npm i https://github.com/hexojs/hexo-renderer-markdown-it.git --save
```

添加其他扩展:

```
npm i markdown-it-task-checkbox --save
```



更改配置文件[^4] [^5]：

```
markdown:
 render:
   html: true
   xhtmlOut: false
   breaks: true
   linkify: true
   typographer: true
   quotes: “”‘’
 plugins: #要注意下面的缩进
    - markdown-it-abbr
    - markdown-it-footnote
    - markdown-it-ins
    - markdown-it-sub
    - markdown-it-sup
    - markdown-it-deflist
    - markdown-it-imsize #我没有安装,一个自定义图片大小的插件
    - markdown-it-mark
    - markdown-it-regexp #不知道做什么的
    - markdown-it-task-checkbox
    - name: markdown-it-container
      options: success
    - name: markdown-it-container
      options: info
    - name: markdown-it-container
      options: warning
    - name: markdown-it-container
      options: danger
    - markdown-it-deflist
    - name: markdown-it-emoji
      options:
        shortcuts: {}
 anchors:
   level: 2
   collisionSuffix: ''
```



目前大部分都完成的很好:

{% asset_img 1574393274947.png %}

### [修改链接][]

安装 [**hexo-abbrlink**][abb]插件

```
npm install hexo-abbrlink --save
```

修改根目录下的配置文件:

``` yml _config.yml
permalink: :abbrlink/
#abbrlink配置
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: dec    # 进制：dec(default) and hex
```



之后的md中会自动添加:

```
abbrlink: xxxx
```





---

[^1]:[hexo-renderer-markdown](https://github.com/niemingzhao/hexo-renderer-markdown)
[^2]:https://github.com/hexojs/hexo-renderer-markdown-it/wiki/Advanced-Configuration
[^3]:https://huixisheng.github.io/npm-install/
[^4]:https://www.wuliang.me/markdown-it-in-hexo/
[^5]:https://titangene.github.io/article/hexo-markdown-it.html

[修改链接]:https://zuiyu1818.cn/posts/NexT_seourl.html "更改永久链接的默认方式"
https://github.com/rozbo/hexo-abbrlink