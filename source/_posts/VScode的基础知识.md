---
title: VScode的基础知识
toc: true
mathjax: true
tags:
  - VScode
categories:
  - 编程
abbrlink: 33118b3d
date: 2018-12-03 15:33:22
---

# VS Code

在windows中使用python编程，一直对与环境的搭建和控制产生了迷茫。有人说用PyCharm，WinPython中自带了Spyder，在我看来都很不错，却不是够精致。我也在Notepad++上写过东西，用起来也很不错。这次，我打算尝试VS Code。很漂亮，这就够了。

- python的虚拟环境，接触了pipenv，结合MSYS2搭建。

- 结合win10的特性，是否可以在ubuntu for windows上搭建编辑环境？

<!--more-->

## Get Started

### Overview[^1]

VS code 轻量、开源，有众多实用插件、跨平台。

#### 热门扩展

排名不分先后

1. python
2. C/C++
3. ESLint
4. C#
5. Debugger for Chrome
6. vscode-icons
7. TSLint
8. Language Support

### Setup[^2]

Windows 下可安装也可使用绿色包（zip），使用压缩包的时候，注意关闭CODE的自动更新。

附加组件与扩展。能够完善IDE的功能，添加第三方的工具

### Additional Components and tools

#### 常用组件

- git	VS Code内建了代码管理，但仍需要Git组件的安装

#### 热门扩展

眼熟不？

#### 附加工具

能够增强体验的工具链：

- Yeoman
- generator-aspnet
- generator-hottowel
- Express
- Gulp
- Mocha
- Yarn

据说大部分的tools都需要Node.js和npm

### User interface

- `fiels.exclude`setting 设置不考虑的文件及文件夹，比如`.git`


### 更多技巧：

https://code.visualstudio.com/docs/getstarted/tips-and-tricks

## User Guide

### Basic Editing

键盘的快捷键可以根据习惯使用熟悉软件的映射，修改也很方便。

#### 多选与多光标

多光标，就是可以在多个光标处同时输入与修改。

Alt+Click

Ctrl+Alt+Down or Ctrl+Alt+Up 能够快速复制当前行

Ctrl+shift+L 添加文件中相同选择内容到多光标。（用起来很方便）

##### 相应设置：

`editor.multiCursorModifier`

##### 代码块选择

Shift+Alt+Left

Shift+Alt+Right

#### 列选择

shift+alt和鼠标拖画

Ctrl+Shift+Alt+Down/Up/Left/Right/PageDown/PageUp

#### 保存和自动保存

#### Hot Exit

保持工作内容，不保存退出。

#### 跨文件搜索

Ctrl+shift+F

#### 代码格式化

shift+alt+f

ctrl+k ctrl+f

#### 代码折叠

shift+Click

Ctrl+shift+[/]

#### 缩进

一般`tab`是4个空格，可以通过`editor.insertSpaces`和`editor.tabSize`修改

#### 文件编码

`files.encoding`



----
[^1]: https://code.visualstudio.com/docs 

[^2]: [https://code.visualstudio.com/docs/setup/setup-overview](https://code.visualstudio.com/docs/setup/setup-overview)