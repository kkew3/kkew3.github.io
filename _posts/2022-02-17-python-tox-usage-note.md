---
layout: post
title:  "Python Tox 使用笔记"
date:   2022-02-17 19:07:47 +0800
tags:   dev--python
---

[Tox](https://tox.wiki/en/latest/index.html)是一个项目自动化工具，在此记录下没在文档和网上tutorial找到的使用细节。试验中尽可能使用最小`tox.ini`。本文使用`tox --showconfig -- <args...>`的形式观察配置结果。如果文中没有提`<args...>`是什么（例如直接说“配置结果为”，而不是“运行...后配置结果为“），那么运行的是`tox --showconfig`。

## 默认`basepython`

### 情况一
令`tox.ini`为空。此时只有一个匿名虚拟环境。

配置结果为：

```
...

[testenv:python]
...
basepython = /Library/Frameworks/Python.framework/Versions/3.9/bin/python3
...
```

这里的`/Library/Frameworks/Python.framework/Versions/3.9/bin/python3`是本机上按`PATH`顺序第一个遇到的Python解释器（注意这里既不是第一个`python`也不是第一个`python3`）。另外可以观察到，匿名虚拟环境被命名为`python`。

### 情况二

令`tox.ini`为

```ini
[testenv:x]
```

此时只有一个名为`x`的虚拟环境，`x`不与[文档](https://tox.wiki/en/latest/config.html#tox-environments)中的任何一种特殊命名匹配。配置结果为

```
...

[testenv:x]
...
basepython = /Library/Frameworks/Python.framework/Versions/3.9/bin/python3
...
```

可见与情况一相同。

### 情况三

令`tox.ini`为

```ini
[testenv:py28]
```

此时只有一个名为`py28`的虚拟环境。配置结果为

```
...

[testenv:py28]
...
basepython = python2.8
...
```

我们知道是没有`python2.8`的，可见`tox`这里只是做了一个简单的从`pyMN`到`pythonM.N`的映射。此时如果运行`tox`的话是要报错的（即使`tox.ini`里加上`skipsdist = true`也会报错）：`ERROR: InterpreterNotFound: python2.8`。

### 情况四

令`tox.ini`为

```ini
[testenv:py28]
basepython = python2.7
```

与情况三相同，但显式指定了`basepython`。配置结果为

```
...

[testenv:py28]
...
basepython = python2.7
...
```

可见显式指定的`basepython`生效了。

## `{posargs}`展开

### 情况一

令`tox.ini`为

```ini
[testenv]
commands = {posargs}
```

运行`tox --showconfig`后（无参数），配置结果为

```
...
commands = [[]]
...
```

可见`{posargs}`在无参数时展开为空字符串。

运行`tox --showconfig -- hello world`后（带参数），配置结果为

```
...
commands = [['hello', 'world']]
...
```

在`{toxinidir}`下新建两个文件`hello1`和`hello2`，然后运行`tox --showconfig -- hello*`后（注意这里的运行环境不是Windows），配置结果为

```
...
commands = [['hello1', 'hello2']]
...
```

这是符合期望的，因为Shell在传参前先做了Globbing，然而如果运行`tox --showconfig -- "hello*"`后，配置结果为

```
...
commands = [['hello*']]
...
```

可见`{posargs}`不会做Globbing。

举一个运行`tox`的例子。令`tox.ini`为

```ini
[tox]
skipsdist = true

[testenv]
allowlist_externals = ls
commands = ls {posargs}
```

如果运行`tox -- "hello*"`，我们会得到结果

```
python run-test-pre: PYTHONHASHSEED='2558120981'
python run-test: commands[0] | ls 'hello*'
ls: hello*: No such file or directory
ERROR: InvocationError for command /bin/ls 'hello*' (exited with code 1)
_________________________ summary __________________________
ERROR:   python: commands failed
```

### 情况二

令`tox.ini`为

```ini
[testenv]
commands = "{posargs}"
```

注意`{posargs}`两边的引号。运行`tox --showconfig`后（无参数），配置结果为

```
...
commands = [['']]
...
```

可见虽然`{posargs}`在无参数时展开为空字符串，但现在有引号，导致仍产生了一个参数，只不过该参数值为空。

运行`tox --showconfig -- hello`后（一个参数），配置结果为

```
...
commands = [['hello']]
...
```

没什么值得惊讶的。

运行`tox --showconfig -- hello world`后（多参数），配置结果为

```
...
commands = [['hello world']]
...
```

可见虽然`{posargs}`展开成了两个参数，但是引号又重新把它们括成了一个参数。
