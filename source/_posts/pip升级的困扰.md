---
title: pip升级的困扰
toc: true
mathjax: true
tags:
  - - python
    - pip
categories:
  - python
abbrlink: 926935ee
date: 2018-12-06 17:38:37
---

# pip该升级了

！pip该升级了，系统总是提示新版本已经是18了， 我还在用10。其实我觉得还不错，因为可以很方便的通过`pip -V`指令看出我是不是在`pipenv shell`环境里。

然后，就是手贱了。

哦,对了,我是windows

<!--more-->

### 001
```
>pip install --upgrade pip
 ERROR: To modify pip, please run the following command:
  x:\soft\python364\python.exe -m pip install --upgrade pip
```

哦,原来是这个命令

```
>python -m pip install --upgrade pip
Looking in indexes: https://pypi.tuna.tsinghua.edu.cn/simple
Collecting pip
 Using cached https://pypi.tuna.tsinghua.edu.cn/packages/c2/d7/90f34cb0d83a6c5631cf71dfe64cc1054598c843a92b400e55675cc2ac37/pip-18.1-py2.py3-none-any.whl
Installing collected packages: pip
 Found existing installation: pip 10.0.1
   Uninstalling pip-10.0.1:
     Successfully uninstalled pip-10.0.1
 Rolling back uninstall of pip
Exception:
Traceback (most recent call last):
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\basecommand.py", line 228, in main
   status = self.run(options, args)
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\commands\install.py", line 335, in run
   use_user_site=options.use_user_site,
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\req\__init__.py", line 49, in install_given_reqs
   **kwargs
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\req\req_install.py", line 748, in install
   use_user_site=use_user_site, pycompile=pycompile,
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\req\req_install.py", line 961, in move_wheel_files
   warn_script_location=warn_script_location,
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\wheel.py", line 431, in move_wheel_files
   generated.extend(maker.make(spec))
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_vendor\distlib\scripts.py", line 403, in make
   self._make_script(entry, filenames, options=options)
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_vendor\distlib\scripts.py", line 307, in _make_script
   self._write_script(scriptnames, shebang, script, filenames, ext)
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_vendor\distlib\scripts.py", line 243, in _write_script
   launcher = self._get_launcher('t')
 File "X:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_vendor\distlib\scripts.py", line 382, in _get_launcher
   result = finder(distlib_package).find(name).bytes
AttributeError: 'NoneType' object has no attribute 'bytes'
```

em~,似乎出了个什么错误?

不过升级完了吗?

```
>pip -V
 pip 10.0.1 from x:\soft\python364\lib\site-packages\pip-10.0.1-py3.6.egg\pip (python 3.6)

>python -m pip -V
 pip 18.1 from X:\soft\python364\lib\site-packages\pip (python 3.6)
```

有两个版本

```
 python -m pip uninstall pip-10
 Skipping pip-10 as it is not installed.
```

### 002

老版本应该已经卸载完了，那么，手贱*2，我删除了pip-10的那个文件夹。

```
>pip
 Traceback (most recent call last):
   File "x:\soft\python364\lib\site-packages\pkg_resources\\\__init\_\_.py", line 659, in _build_master
     ws.require(\_\_requires\_\_)
   File "x:\soft\python364\lib\site-packages\pkg_resources\\\_\_init\_\_.py", line 967, in require
     needed = self.resolve(parse_requirements(requirements))
   File "x:\soft\python364\lib\site-packages\pkg_resources\\\_\_init__.py", line 858, in resolve
     raise VersionConflict(dist, req).with_context(dependent_req)
 pkg_resources.VersionConflict: (pip 18.1 (x:\soft\python364\lib\site-packages), Requirement.parse('pip==10.0.1'))

 During handling of the above exception, another exception occurred:

 Traceback (most recent call last):
   File "X:\soft\python364\Scripts\pip-script.py", line 6, in <module
     from pkg_resources import load_entry_point
   File "x:\soft\python364\lib\site-packages\pkg_resources\__init__.py", line 3017, in <module
     @_call_aside
   File "x:\soft\python364\lib\site-packages\pkg_resources\__init__.py", line 3003, in _call_aside    f(*args, **kwargs)
   File "x:\soft\python364\lib\site-packages\pkg_resources\__init\_\_.py", line 3030, in _initialize_master_working_set
     working_set = WorkingSet._build_master()  File "x:\soft\python364\lib\site-packages\pkg_resources\\\_\_init\_\_.py", line 661, in _build_master
     return cls._build_from_requirements(\_\_requires\_\_)
   File "x:\soft\python364\lib\site-packages\pkg_resources\__init\_\_.py", line 674, in _build_from_requirements    dists = ws.resolve(reqs, Environment())
   File "x:\soft\python364\lib\site-packages\pkg_resources\\\_\_init__.py", line 853, in resolve
     raise DistributionNotFound(req, requirers)pkg_resources.DistributionNotFound: The 'pip==10.0.1' distribution was not found and is required by the application
```

好吧，`pip`命令目前不能直接执行了， `python -m pip`还可以。该怎么做？

### 003

我打算去看看报错的那些文件

首先

```
  File "x:\soft\python364\lib\site-packages\pkg_resources\\\__init\_\_.py", line 659, in _build_master
 ​    ws.require(\_\_requires\_\_)
```

嗯。完全不是能搞定的事情，我还是老老实实的找安装包吧

### 004

1. https://pypi.org/project/pip/#files下载`pip-18.1.tar.gz`
2. 解压放在`site-packages`里，并进入文件夹
3. 执行`python setup.py install`

搞定