---
title: python基础
toc: true
mathjax: true
tags:
  - python
categories:
  - python
abbrlink: 81b2f4bf
date: 2018-12-26 14:48:35
---

# python学习笔记（一）

## 基础知识

[TOC]

参考学习：

1.  [廖雪峰python教程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431664106267f12e9bef7ee14cf6a8776a479bdec9b9000)
2. 《python核心编程（第二版）》第二章

### 注释

`#`为单行注释。有时使用`'''`范围内的多行字符串做多行注释。而且多行字符串还能够通过什么方法提出变成文档。

<!--more-->

### 操作符

python中有标准算术操作符：

`+  -  *  /  //  %  **`

比较操作符：

`<  <=  >  >=  ==  !=` 

逻辑操作符：

`and  or  not`

#### 除法

python的除法有些复杂。

1. `/`

在python 2.x中/除法就跟我们熟悉的大多数语言，比如Java啊C啊差不多，整数相除的结果是一个整数，把小数部分完全忽略掉，浮点数除法会保留小数点的部分得到一个浮点数的结果。

在python 3.x中/除法不再这么做了，对于整数之间的相除，结果也会是浮点数。

python 2.x:

```python
>>> 1/2
0
>>> 1.0/2.0
0.5
```

python 3.x:

```python
>>> 1/2
0.5
```

2. `//`

`//`叫做floor除法，在python 2.x 和 3.x中，表现一致

python 2.x:

```python
>>> -1//2
-1
```

python 3.x:

```python
>>> -1//2
-1
```

注意的是并不是舍弃小数部分，而是执行 floor 操作，如果要截取整数部分，那么需要使用 math 模块的 trunc 函数

python 3.x:

```python
>>> import math
>>> math.trunc(1/2)
0
>>> math.trunc(-1/2)
0
```

#### 不等号

python 2.x时代不等于有两个比较操作符，`!=`和`<>`，分别是 C 风格和 pascal 风格。目前`<>`已经被淘汰。

#### 连续比较

```python
>>> 3 < 4 <5
True
>>> 3 < 4 and 4 < 5
True
```

### 数字类型

python支持5种基本数字类型：

- int
- long
- bool
- float
- complex （复数）
- decimal（十进制浮点）

关于decimal，它不是内建类型，需要导入decimal模块。举例来说，由于二进制表示数字`1.1`的时候，有一个无限循环片段，无法精确表示。

```python
>>> 1.1
1.1000000000000001
>>> print(decimal.Decimal('1.1'))
1.1
```

### 字符串和编码

Unicode把所有语言的编码都统一到一套代码里，这样就不会再有乱码的问题了。随着Unicode标准的不断发展，现代操作系统和大多数的编程语言都直接支持Unicode。

但Unicode表示常见字符需要两个字节，偏僻字符甚至需要4个字节，如果文本基本上都是英文的话，用Unicode比用ASCII编码要浪费已被的存储空间，在**存储**和**传输**上十分的不划算。于是出现了“可变长编码”的**UTF-8**

**UTF-8**根据字符不同编码成1 ~ 6个字节。常用英文字母1个字节（同ASCII码），汉字通常3字节，生僻字符4~6字节。如果使用大量英文字符，UTF-8就很节省空间。

**计算机通用字符编码工作方式**：

内存中统一使用UNICODE，当需要保存到硬盘或需要传输的时候，就转换为UTF-8。

#### python中的字符串

python 3中，默认以Unicode编码。提供`ord()`函数获取字符的整数表示，`chr()`则把编码转为对应字符。

```python
>>> ord('A')
65
>>> ord('中')
20013
>>> chr(66)
'B'
>>> chr(25991)
'文'
>>> '\u4e2d\u6587' #十六进制表示方法
'中文'
```

Python的字符串类型是`str`，在内存中以Unicode表示。可以通过`encode()`和`decode()`转换编码：

```python
>>> 'ABC'.encode('ascii')
b'ABC'
>>> '中文'.encode('utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'
>>> '中文'.encode('ascii')  #中文超出了ascii 的编码范围，所以会报错
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)

>>> b'ABC'.decode('ascii')
'ABC'
>>> b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
'中文'
```

如果`bytes`中包含无法解码的字节，`decode()`方法会报错：

```python
>>> b'\xe4\xb8\xad\xff'.decode('utf-8')
Traceback (most recent call last):
  ...
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in position 3: invalid start byte
```

如果`bytes`中只有一小部分无效的字节，可以传入`errors='ignore'`忽略错误的字节：

```python
>>> b'\xe4\xb8\xad\xff'.decode('utf-8', errors='ignore')
'中'
```

保存文件的时候，声明utf-8编码：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```

#### 格式化显示

同 C 语言。

| 占位符 | 内容         |
| ------ | ------------ |
| %d     | 整数         |
| %f     | 浮点数       |
| %s     | 字符串       |
| %x     | 十六进制整数 |

当不知道应该选用什么的时候，`%s`永远起作用。

其中，格式化整数和浮点数还可以指定是否补0和整数与小数的位数：

 ```python
print('%2d-%02d' % (3, 1))
print('%.2f' % 3.1415926)
 ```

#### format()

另一种格式化字符串的方法是使用字符串的`format()`方法，它会用传入的参数依次替换字符串内的占位符`{0}`、`{1}`……，不过这种方式写起来比%要麻烦得多：

```python
>>> 'Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125)
'Hello, 小明, 成绩提升了 17.1%'
```

### 列表、元组、字典

[]

()

{}

