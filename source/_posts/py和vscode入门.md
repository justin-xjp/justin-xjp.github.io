---
title: py和vscode入门
toc: true
mathjax: true
tags:
  - python
  - VScode
categories:
  - 编程
abbrlink: 3391ae3a
date: 2018-12-07 15:36:28
---

来自于VScode 的文档
# Getting Start with Python in VS CODE

教程中会通过创建一个简单的“hello world”来熟悉使用扩展。

### 前提条件

需要如下准备：

1. 安装python 扩展。

2. 安装python3。

3. 在 windows 上确保`path`系统环境变量包含了解释器路径。

<!--more-->

## 基本操作

- 打开工作区文件夹
- 选择python interpreter
- 创建hello world代码文件
  - 输入代码的时候能够注意到代码自动补全功能，该功能会覆盖所有标准模块，和当前环境已安装模块。同时还提供了对象类型上可用的方法。
- 运行程序
  - 右键——在终端中运行代码
  - 选中代码，`shift+enter`，相当于在终端中运行选中的代码
  - `Python: Start REPL`，打开一个终端，一次一行代码

### 配置并运行debugger

- `f9`设置断点，也可以在行号附近直接鼠标点击。
- 选择debug试图
- 打开配置菜单，会创建`launch.json`
- 选择**Python: Current File (Integrated Terminal)**
  - 添加`"stopOnEntry": true`，使得程序自动在第一行停下来。
- 在代码页面`f5`执行，因为刚刚设设置，debug在第一行前停下来。
- 调试栏按钮：继续（`F5`）、跨步（`F10`）、进入（`F11`）、退出（`Shift+F11`）、重新启动（`Ctrl+Shift+F5`）和停止（`Shift+F5`）
- 更详细的信息看[debug配置](https://code.visualstudio.com/docs/python/debugging)

> **Tip:使用 Logpoints(日志点)输出过程信息**: 开发过程中经常使用`print`语句快速输出过程中信息。在VS Code中可以使用`Logpoints`替代。`Logpoint`就像一个断电，它会将消息记录到控制台，且不会停止该程序。更多信息参考[Logpoints](https://code.visualstudio.com/docs/editor/debugging#_logpoints)

### 排除故障

某些原因没有自动生成`launch.json`，可以自行创建。

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File (Integrated Terminal)",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal"
        }
    ]
}
```

### 安装和使用包

来个更有趣的例子。本例中将使用`matplotlib`和`numpy`创建图形，就像是数据科学常用的那样。（注意，win10的WSL是没有UI的，不能成功运行`matplotlib` ）

- 创建`standardplot.py`文件并添加以下代码：

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 20, 100)  # Create a list of evenly-spaced numbers over the range
plt.plot(x, np.sin(x))       # Plot the sine of each x point
plt.show()                   # Display the plot
```

尝试运行代码，然后你会看到"ModuleNotFoundError: No module named 'matplotlib"，提醒你没有安装matplotlib模块。

- 运行`Terminal: Crate New Integrated Terminal(ctrl +shift + ')`,创建一个新的集成终端。然后输入以下命令：

```powershell
# Don't use with Anaconda distributions because they include matplotlib already.

# macOS
sudo python3 -m pip install matplotlib

# Windows (may require elevation)
python -m pip install matplotlib

# Linux (Debian)
sudo apt-get install python3-tk
python -m pip install matplotlib

# Windows + pipenv 
pipenv install matplotlib
```

- `matplotlib`和`numpy`（作为依赖）安装成功后，再次运行程序。有一个包含图像绘图窗口出现

**debug**里没有找到虚拟环境下的`matplotlib`。尝试查看**[debug设置](https://code.visualstudio.com/docs/python/debugging)**

---



# Python和VS Code

## 环境搭建问题及解决

我的生产环境，python和vs code都是在VHD里，并且是绿色解压安装。我需要解决以下问题：

- [ ] ~~每次对VHD进行挂载/卸载时，对应添加/删除PATH中的变量。~~

     目前来看，这个想法不切实际。应该是涉及到注册表操作，而且还要考虑在操作过程中，环境变量发生改变该如何更新。

- [x] 最好不用修改PATH即可直接使用。无可避免的还是设置了，而且设置了更多的系统变量。

- [x] 用好pipenv或其他环境设置

- [x] 修改不同env时，python 包的安装位置

- [x] VS CODE在发现缺少包，并自动执行PIP安装时，安装到了`%appdata%\python\python36\`下。问题出在VSCODE在使用 pip install的时候添加了--user参数，使得安装到了上述文件夹，需要继续学习CODE的配置。

### 成功1

安装VS code和python，不改变默认路径，成功了。

### 绿色解压在虚拟盘

因为在折腾中瞥间过一次成功。现在一个一个改

#### VS Code portable

在解压目录添加`data`子文件夹，升级时保留这个文件夹不被替换。

添加`tmp`子文件夹.

**portable**模式与安装版可以同时存在，安装版能够找到pipenv的interpreter，portable的找不到，与下面的结果相同。这应该是个bug。

#### step1

1. 添加vs code的路径到path。使用powershell仍然不能使用命令行。
2. 添加python 和 python\script 到path

#### step2

- **找不到pylint**，更改默认`pip install --user`的路径。在系统环境变量中添加：`PYTHONUSERBASE=/XXXX/`。可以指定--user的目标目录。同时添加系统环境变量`PYTHONPATH`，将`%PYTHONUSERBASE%/PYTHON36/SCRIPTS`添加进去，使**python 能够找到它**。添加此项**不需重启**
- **对应上不上的.ven环境文件**。
  - 直接pipenv install。文件建立在`c:\user\name\.virtualenvs\`下。不能列出选择的虚拟interpreter。
    - 添加`c:\user\name\.virtualenvs\xxxx\python.exe`到`python.pythonPath`属性。出现过一次可以选择，然后就又没有了。**重启试试**。仍然不行。

最后的体验：

- vs code是能够认出pipenv环境的，当弹出需要安装pylint的时候，会使用`pipenv install pylint -dev`的命令。但是人不能提供`interpreter pipenv`的选择。不久前看到的成功是在设置了`python.pythonPath`的情况下出现的，而且不能再现。
- 尝试设置`python.venvPath`指向`x:\xxxx\.virtualenvs\`，没有任何效果。

**至此，U盘 (或是VHD) 的 python + vs code 尝试失败**， 要是有人有能力实现，烦请告知。其实，我认为主要是pipenv的支持出现了问题，如果换成其他方式的虚拟环境，应该是不成问题的。

### 其他设置

添加了` "git.path":"X:\\soft\\git\\cmd\\git.exe",`到用户设置文件，settings.json。git 就可以用了。手动时期git都是用的ubuntu里的，现在常用的缩写st, ci, lg, 什么的都不能使用，但相对来说，Code提供了图形化的方式，也更方便了。
唯一的问题：*X* 是VHD挂载的硬盘，虽说现在没有问题，但不能保证什么时候都会出现在 **X** 这个位置上。该怎么解决？

- 在CODE的快捷方式添加了`X:\soft\VSCode-win32-x64\Code.exe --extensions-dir X:\soft\vscodeexten\`，将扩展安装位置移到了X目录下，正尝试移动默认配置文件存储位置。
- 添加`WORKON_HOME=%home%\.virtualenvs`和`home=x:\xxx`到系统变量，可改变pipenv（实际是virtualenv）环境文件夹的位置。
- 添加`x:\python3.6\script`到`path`变量，使VS CODE能够找到pipenv。
- **在`.vscode\settings.json`中添加`"python.venvPath": "X:\\xxxx\\.virtualenvs\\xxxxxx",`，尝试环境的人工指定。VScode能自动识别PIPENV并挂在虚拟环境。仅识别一个pipenv就不再继续（其实，我也只会用一个目前)**
- 在`%appdata%\pip\pip.ini`中添加如下，可以改变PIP的服务器。

> [global]
> index-url = https://pypi.tuna.tsinghua.edu.cn/simple

- 在使用pipenv的环境中不能存在中文路径。
- 在`pipfiles`里更改如下，可在环境中使用pypi镜像站：

> [[source]]
>
> url = "https://pypi.tuna.tsinghua.edu.cn/simple"

- `pipenv run`和`pipenv shell`是自动识别目录下`.env`文件并加载的。
- 在[vscode-python-issues](https://github.com/Microsoft/vscode-python/issues)里找东西比一行行看手册快多了。

## Using Python environments in VS Code

一般情况下，python扩展会默认寻找path变量中的python解释器，如果没有找到，每次启动都会报错。着对我绿色安装无疑是受打击的。设置`python.disalbeInstallationCheck`为`ture`能够关闭这个检测。

### 选择并激活一个环境。

> python:Select Interpreter

你可以随时切换环境。命令执行时会显示默认环境、conda环境和virtual 环境。

python扩展使用选择的环境来运行代码（使用命令python:run python file in terminal），当你打开`.py`文件时，会提供语言服务（自动补全，查错，语法检查，格式化等）。使用`terminal:creat new integrated terminal`会自动创建环境。（看起来并没用运行虚拟环境PIPENV SHELL。是哪里理解有错吗？）

状态栏会一直显示当前选用的解释其，点击它就可以快速执行`python:select interpreter`命令。

#### 环境和终端窗口

在选择一个解释器（环境）后，右键点击列表中的文件，并选择**在终端中运行python文件**。环境也会自动激活。除非修改`python.erminal.activateEnvironment`为`false`。

然而，在SHELL里运行VS CODE，并不会自动激活虚拟环境，需要使用`terminal:creat new integrated terminal`。还要注意powershell中运行vs code，不会激活conda环境。

在任何已经打开的环境终端中进行修改都是可行的。并且切换环境并不会影响已经打开的环境。你可以切换不同的环境打开不同的终端。

#### 选择debugging环境

`python.pythonPath`（注意P的大写）设定了debugging用的环境。此变量在select interpreter命令后会自动改变。此变量可同时存在于`launch.json`和workspace setting以及`user settings.json`。他的选取优先顺序为：

1. launch.json
2. Workspace
3. User settings.json

### 环境文件的位置

扩展自动在以下位置寻找解释器（interpreters）：

- 常规安装文件路径：`/usr/local/bin`,`/usr/sbin`,`/sbin`,`c:\\python27`,`c:\\python36`
- 在workspace目录下的虚拟环境目录
- 由`python.venvPath`指定的虚拟环境，可以含有多个环境，指定总目录，扩展会搜寻一级子目录。（select interpreter命令仍然只能看到一个默认解释器，我的pipenv找不到呢？）
- pyenv安装的解释器
- 一个对应workpalce文件夹的pipenv环境，如果找到一个，就不会再寻找其他解释器环境，或是pipenv期望管理的环境。

仍然可以手动指定环境，如果没有认出来的话.

#### 全局、虚拟 和conda 环境变量

默认情况下，任何的经你安装的python interpreter都会运行他自己的全局环境（global environment)，并不局限于特定的某个项目 。例如，如果你只是在命令行下运行Python，则实在全局环境的解释器中运行的。因此，安装或卸载任何包都会影响到全局环境及在全局环境中运行的程序。（python扩展的2018.8.1版及以后，会自动更新运行环境）

尽管在全局环境中工作是一种简单的入门方法，但随着时间的推移，运行环境将因为你为不同的工程安装的大量不同的包（甚至有冲突）而变得杂乱无章。这种杂乱使得很难针对一组特定版本的包对程序进行彻底测试，比如生产环境或服务器环境。

为此，开发人员通常会创建一个虚拟环境。虚拟环境是项目中包含特定解释器副本的子文件夹。当你激活虚拟环境时，安装的任何包都只存在于该虚拟环境的子文件夹中。然后，当你在该环境中运行python程序时，你就知道它在这些特定包下运行的状况。

> Tip: conda 环境时使用 conda包管理器进行创建和管理的。

__创建虚拟环境__，使用如下命令，其中".venv"时环境文件夹的名称：

```powershell
# Windows
python -m venv .venv
#或是py -3 -m venv .venv
```

如何在项目中使用虚拟环境的示例，参见https://code.visualstudio.com/docs/python/tutorial-django 和https://code.visualstudio.com/docs/python/tutorial-flask

**到这里，原页面教程中关于虚拟环境一直使用的是venv，没有更现代的工具。**

> 在早期的（2018.10前）python扩展中，如果你在vs code的terminal里改变或创建了一个虚拟环境，需要在`command palette`执行**Reload Window**命令，然后用**Python:Select Interpreter**命令激活环境。
>
> `requirements.txt`文件可以帮助重建需要的环境，`pip freeze > requirements.txt`生成文件，使用`pip install -r requirements.txt`再次部署。通过在不同的虚拟环境还原requirements文件的描述来达到复制环境的目的。这个做法好原始。

##### 手动指定解释器

1. 可以在工作区域（workspace）中的settings中添加`python.pythonPath`属性

```
	"python.pythonPath": "c:/python36/python.exe"
```

2. 还可以使用`python.pythonPath`指向虚拟环境

```
	"python.pythonPath": "c:/dev/venv/Scripts/python.exe"
```

3. 或是使用环境变量。如创建一个`PYTHON_INSTALL_LOC`的变量并在设置中：(经试验，*不好用* )

```
	"python.pythonPath": "${env:PYTHON_INSTALL_LOC}"
```

环境变量在WIN10环境中没有成功。但这应该是在不同系统间传递项目的最简单设置，只需在系统上设置环境变量即可。

### 虚拟环境定义文件

这个文件是一个简单的文本文件，其中键值、变量一行一个，不支持多行变量。`environment_variable=balue`可用`#`注释。

默认情况下，python扩展会加载当前工作区中的`.env`文件，这个文件默认由`"python.envFile": "${workspaceFolder}/.env"指定。可是随时改变这个设定成任意其他文件名。（但不建议更改）

debug环境同样包含一个`envFile`属性，也默认为当前工作区的`.env`。可修改成调试目的用的变量。

举例来说，当开发一个web应用时，可能需要在开发服务器和生产环境之间来回切换。就可以如下操作：

**dev.env**

``` shell
#dev.env - development configuration

#API endpoint
MYPROJECT_APIENDPOINT=https://my.domain.com/api/dev/

# Variables for the database
MYPROJECT_DBURL=https://my.domain.com/db/dev
MYPROJECT_DBUSER=devadmin
MYPROJECT_DBPASSWORD=!dfka**213=
```

**prod.env**

```shell
# prod.env - production configuration

# API endpoint
MYPROJECT_APIENDPOINT=https://my.domain.com/api/

# Variables for the database
MYPROJECT_DBURL=https://my.domain.com/db/
MYPROJECT_DBUSER=coreuser
MYPROJECT_DBPASSWORD=kKKfa98*11@
```

然后，就还可以更改`python.envFile`指向`${workspaceFolder}/prod.env`, debug环境的`envFile`属性指向`${workspaceFolder}/dev.env`

### 使用PYTHONPATH环境变量

pythonpath环境变量，指导interpreter去查找模块应该搜寻的其他位置。pythonPath可以包含多个路径，由`os.pathsep`分离（WINDOWS是分号，linux是冒号），无效的路径会被忽视。

在VS CODE中，pythonPath会影响debug，linting，代码自动补全，单元测试以及其他依赖于python解析模块的操作。例如，你有一些源代码在`src`目录，测试在`tests`目录。当你运行测试时，它不能正常地访问到`src`中的模块，除非将`src`添加到PYTHONPATH中。

建议在环境变量定义文件中设置PYTHONPATH。

> **Note**: PYTHONPATH **不指定**python interpreter的路径，所以**永远不要**用它当`python.pythonPath`使用。这个命名很糟糕，容易带来误解，c'est la vie。所以多阅读[PYTHONPATH文档](https://docs.python.org/3.7/using/cmdline.html#envvar-PYTHONPATH)几次，并且在脑子里确定，PYTHONPATH不是用来指向interpreter的。