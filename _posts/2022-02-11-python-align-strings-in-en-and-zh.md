---
layout: post
title:  "如何在Python中对齐中英文混排字符串"
date:   2022-02-11 14:27:19 +0800
tags:   dev--python
---

Python中有`str.ljust`、`str.rjust`、`str.center`用于左对齐、右对齐和居中对齐字符串。例如`'hello'.ljust(10, '*')`返回`'hello*****'`，`'hello'.rjust(10, '*')`返回`'*****hello'`，等。每个中日韩文（CJK字符）在Python中被视为一个字符，然而它们的显示宽度为2，这个矛盾使`ljust`、`rjust`、`center`不能正确地对齐CJK字符：例如`'你好'.ljust(5, '*')`返回`'你好***'`而不是`'你好*'`。另见[此文](https://blog.csdn.net/qq_45537774/article/details/99727637)。

为了阐述如何解决这个问题，假设我们要以$w$显示宽度对齐字符串`s`，并以`ljust`([doc](https://docs.python.org/3/library/stdtypes.html#str.ljust))为例（另外两个同理），另假设`fillchar='*'`。易知我们需要在`s`的右侧补$w-l$个`'*'`，其中$l$是`s`的显示宽度。而为了使`ljust`为我们补$w-l$个`'*'`，`ljust`的第1个参数应为$n+w-l$，其中$n$为`s`的字符数。做简单的变换：$n+w-l = w-(l-n)$。假设`s`中有$a$个显示宽度为1的字符、$b$个显示宽度为2的字符，则$l=a+2b$，$n=a+b$，因此$l-n=b$，即$n+w-l=w-b$。如果`s`中显示宽度为2的字符限于CJK字符，那么$b$即为CJK字符的个数。Python中求CJK字符在一个字符串`string`中的个数的函数为：

```python
import unicodedata

def count_cjk_chars(string):
    return sum(unicodedata.east_asian_width(c) in 'FW' for c in string)
```

不难得到适用于可能含有CJK字符的对齐函数：

```python
def cjkljust(string, width, fillbyte=' '):
    """
    左对齐
    
    >>> cjkljust('hello', 10, '*')
    'hello*****'
    >>> cjkljust('你好world', 10, '*')
    '你好world*'
    >>> cjkljust('你好world', 1, '*')
    '你好world'
    """
    return string.ljust(width - count_cjk_chars(string), fillbyte)


def cjkrjust(string, width, fillbyte=' '):
    """
    右对齐
    """
    return string.rjust(width - count_cjk_chars(string), fillbyte)


def cjkcenter(string, width, fillbyte=' '):
    """
    居中对齐
    """
    return string.center(width - count_cjk_chars(string), fillbyte)
```

完整代码参见我的[Gist](https://gist.github.com/kkew3/8bb9aa225a6c82ae5e1a0fa609c9a65a)。

---

也可从[PyPI](https://pypi.org/project/cjkjust/)下载使用。
