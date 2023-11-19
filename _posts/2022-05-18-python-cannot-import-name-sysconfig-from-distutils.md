---
layout: post
title:  "python cannot import name 'sysconfig' from 'distutils'"
date:   2022-05-18 23:02:51 +0800
tags:   dev--python os--ubuntu
---

搜索这个问题时看到了[这篇博客](https://www.jianshu.com/p/49e4bfd61467)，感觉略麻烦。我就想有没有更简单的方式。后来摸索出来了，记在这里。

## 环境

- Python 3.9.12
- Ubuntu 18.04 LTS

## 安装 Python3.9

详见[这篇回答](https://zhuanlan.zhihu.com/p/343237962)。简要转述如下：

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.9
```

## 问题

```bash
python3.9 -m pip -V
```

报错

```
Traceback (most recent call last):
  File "/usr/lib/python3.9/runpy.py", line 188, in _run_module_as_main
    mod_name, mod_spec, code = _get_module_details(mod_name, _Error)
  File "/usr/lib/python3.9/runpy.py", line 147, in _get_module_details
    return _get_module_details(pkg_main_name, error)
  File "/usr/lib/python3.9/runpy.py", line 111, in _get_module_details
    __import__(pkg_name)
  File "/usr/lib/python3/dist-packages/pip/__init__.py", line 29, in <module>
    from pip.utils import get_installed_distributions, get_prog
  File "/usr/lib/python3/dist-packages/pip/utils/__init__.py", line 23, in <module>
    from pip.locations import (
  File "/usr/lib/python3/dist-packages/pip/locations.py", line 9, in <module>
    from distutils import sysconfig
ImportError: cannot import name 'sysconfig' from 'distutils' (/usr/lib/python3.9/distutils/__init__.py)
```

## 解决方法

注意到上文中`ppa:deadsnakes/ppa`里包含`python3.9-venv`，而`venv`显然依赖`pip`。安装`python3.9-venv`便能自动处理好依赖。

```bash
sudo apt install python3.9-venv
```

再看`python3.9 -m pip -V`即可输出正确的

```
pip 9.0.1 from /usr/lib/python3/dist-packages (python 3.9)
```
