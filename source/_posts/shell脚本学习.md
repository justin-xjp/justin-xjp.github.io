---
title: shell脚本学习
toc: true
mathjax: true
tags:
  - linux
  - shell
categories:
  - linux
abbrlink: f26f620f
date: 2019-11-13 09:47:13
---

 

通过阅读其他人的自动脚本，补足自己的linux基本知识。杂记

<!--more-->



再sh脚本中第一行会有`#!/bin/sh`语句告诉系统使用什么位置的shell执行脚本。**#!**占8位，是文件类型标识，告诉系统此文件由后续程序解释执行。

`$(exec abcd)`，可以将命令用**$()**括起来，将执行结果赋值给变量。

`exit 0`，运行结束的出口状态，出口状态**0**为正常，**1~255**为异常。

`2>/dev/null` ，输出转移，将错误信息（2）输出到 **null**

shell中的函数可以写在一行，写在一行内时，以`; `隔离语句，并且在**；**后应有一个‘ ’ 空格。

`test` 和 `[  ]`，是条件测试语句，可测试文件属性，可测试字符串特性。常用在 **if**判定位置。

`mkdir -p` ，**-p**参数确保文件夹存在，如果不存在就创建一个。但也许会有同名文件占据位置？同名的文件。

`lsb_release`命令可以查看系统信息，如命令不存在，尝试看看`/etc/os-release`或`/etc/lsb-release`文件

判断系统32还是64位，`uname -a`，或者`getconf LONG_BIT`。

### 压缩解压

- tar.xz    `tar -Jcf xxxx`/`tar -Jxf xxxx.tar.xz`

### 文件的复制移动和删除

linux下文件的复制、移动与删除命令为：cp，mv，rm
一、文件复制命令cp

```
命令格式：cp [-adfilprsu] 源文件(source) 目标文件(destination)
cp [option] source1 source2 source3 ... directory
```
 参数说明：
    -a:是指archive的意思，也说是指复制所有的目录
    -d:若源文件为连接文件(link file)，则复制连接文件属性而非文件本身
    -f:强制(force)，若有重复或其它疑问时，不会询问用户，而强制复制
    -i:若目标文件(destination)已存在，在覆盖时会先询问是否真的操作
    -l:建立硬连接(hard link)的连接文件，而非复制文件本身
    -p:与文件的属性一起复制，而非使用默认属性
    -r:递归复制，用于目录的复制操作
    -s:复制成符号连接文件(symbolic link)，即“快捷方式”文件
    -u:若目标文件比源文件旧，更新目标文件 

如将/test1目录下的file1复制到/test3目录，并将文件名改为file2,可输入以下命令：
cp /test1/file1 /test3/file2

二、文件移动命令mv

```
命令格式：mv [-fiv] source destination
```

 参数说明：
    -f:force，强制直接移动而不询问
    -i:若目标文件(destination)已经存在，就会询问是否覆盖
    -u:若目标文件已经存在，且源文件比较新，才会更新

如将/test1目录下的file1复制到/test3 目录，并将文件名改为file2,可输入以下命令：
mv /test1/file1 /test3/file2

三、文件删除命令rm

```
命令格式：rm [fir] 文件或目录
```

参数说明：
    -f:强制删除
    -i:交互模式，在删除前询问用户是否操作
    -r:递归删除，常用在目录的删除

如删除/test目录下的file1文件，可以输入以下命令：
rm -i /test/file1

**复制：**

```
CP命令
格式: CP [选项]  源文件或目录   目的文件或目录
选项说明:-b 同名,备分原来的文件
        -f 强制覆盖同名文件
        -r  按递归方式保留原目录结构复制文件

cp -Rf /home/user1/* /root/temp/
将 /home/user1目录下的所有东西拷到/root/temp/下而不拷贝user1目录本身。
即格式为：cp -Rf 原路径/ 目的路径/
```



**移动：**

```
mv ./WorkReport/web.xml ./WorkReport/WEB-INF/
注：移动/WorkReport/web.xml文件到/WorkReport/WEB-INF/

mv /data/new /data/old/
注：移动/data/new 到/data/old/文件夹下
注意点：移动文件夹的话就不要再加 / 了

如果是移动文件夹下的所有文件的话就可以文件夹后面跟上 /* 

mv /data/new/* /data/old/
```

 

 

## 前人经验

在制作通用性自动脚本时，需要考虑很多环境情况，对发行版本的判断，以及对某一信息在不同软件环境下应如何读取，均需要经验总结。


