---
layout: post
title:  "像BSD ls 一样中英文混排字符串(Python3)"
date:   2022-02-05 18:49:30 +0800
categories: python
---

[这里](https://www.cnblogs.com/LiuYanYGZ/p/14901218.html)有一个C语言实现的字符串打印功能。我没细看它支不支持中英文混排。我在此给一个Python3版的支持中英文混排的字符串打印代码。另见我的Gists：[cjkjust](https://gist.github.com/kkew3/8bb9aa225a6c82ae5e1a0fa609c9a65a)，[fmtstrings\_like\_ls](https://gist.github.com/kkew3/3dde88ec52df12d7cc855ffeb2091a7c)。下面的代码和Gists没有本质差别，只是我在下面新加了一点注释、精简了一点无关代码。

## 代码

中英文混排时的对齐函数`cjkljust`：

```python
try:
    # https://f.gallai.re/cjkwrap
    from cjkwrap import cjklen
except ImportError:
    import unicodedata

    def is_wide(char):
        return unicodedata.east_asian_width(char) in 'FW'

    def cjklen(string):
        return sum(2 if is_wide(char) else 1 for char in string)


def cjkljust(string, width, fillbyte=' '):
    """
    >>> cjkljust('hello', 10, '*')
    'hello*****'
    >>> cjkljust('你好world', 10, '*')
    '你好world*'
    >>> cjkljust('你好world', 1, '*')
    '你好world'
    """
    return string.ljust(len(string) + width - cjklen(string), fillbyte)
```

打印函数`pprint`：

```python
import math
import itertools
import shutil


def calc_layout(n_strings, total_width, column_width, width_between_cols):
    # expected_ncols * column_width +
    #     (expected_ncols - 1) * width_between_cols <= total_width
    #
    #   解得 expected_ncols <= (total_width + width_between_cols) /
    #                          (column_width + width_between_cols)
    # 因此 expected_ncols 最大为不等号右边的向下取整
    expected_ncols = math.floor((total_width + width_between_cols) /
                                (column_width + width_between_cols))
    expected_ncols = max(expected_ncols, 1)
    actual_nrows = math.ceil(n_strings / expected_ncols)
    actual_ncols = (n_strings - 1) // actual_nrows + 1
    return actual_nrows, actual_ncols


def pprint(strings, total_width=None, width_between_cols=1, file=None) -> None:
    """
    Pretty print list of strings like ``ls``.
    :param strings: list of strings
    :param total_width: the disposable total width, default to terminal width
    :param width_between_cols: width between columns, default to 1
    :param file: file handle to which to print, default to stdout
    """
    total_width = total_width or shutil.get_terminal_size().columns
    assert total_width >= 1, total_width
    assert width_between_cols >= 1, width_between_cols

    if not strings:
        return

    # column_width: BSD ls 的列宽为所有待打印字符串的最长长度
    column_width = max(map(cjklen, strings))
    nrows, ncols = calc_layout(
        len(strings), total_width, column_width, width_between_cols)
    columns = [[] for _ in range(ncols)]
    for i, s in enumerate(strings):
        columns[i // nrows].append(s)

    for row in itertools.zip_longest(*columns):
        padded_row = (cjkljust(s or '', column_width) for s in row)
        print((' ' * width_between_cols).join(padded_row), file=file)
```
