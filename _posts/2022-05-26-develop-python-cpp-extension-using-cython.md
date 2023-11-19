---
layout: post
title:  "使用Cython为Python开发C++扩展"
date:   2022-05-26 22:19:31 +0800
tags:   dev--python dev--cpp
---

[Cython](https://cython.org)的出现免去了为Python开发C/C++扩展的很多麻烦。本文以一个简单的例子来说明如何为Python开发C++扩展。

例子程序：给定一个列表，把列表的每个元素平方，并返回新列表。用Python实现会是这样：

```python
def square(l):
    return [x * x for x in l]
```

现在我们用C++实现这个函数。根据[Using C++ in Cython](https://cython.readthedocs.io/en/latest/src/userguide/wrapping_CPlusPlus.html#standard-library)，Python列表对应于C++的`std::vector`，因此我们可以用`std::vector`。

\_square.h:

```cpp
#ifndef _SQUARE_H_
#define _SQUARE_H_

#include <vector>

std::vector<double> _square(std::vector<double> &);

#endif
```

\_square.cpp:

```cpp
#include "_square.h"

std::vector<double> _square(std::vector<double> &l)
{
    std::vector<double> res(l.size());
    for (auto i = l.begin(); i != l.end(); ++i) {
        res.push_back(*i * *i);
    }
    return res;
}
```

注意到上文代码文件名和函数名都以下划线开头，这里没有什么特殊规则，只是不让它们与Cython文件名和函数重名。接下来我们写封装C++的Cython代码。Cython代码后缀是`.pyx`。

square.pyx:

```python
from libcpp.vector cimport vector

cdef extern from "_square.h":
    vector[double] _square(vector[double] l)

def square(l):
    cdef vector[double] l_vec = l
    return _square(l_vec)
```

最后我们编写用于编译的`setup.py`，`setup.py`位于项目根目录。这里假设上述`_square.h`、`_square.cpp`、`square.pyx`都位于Python package `package1.package2`下。

setup.py:

```python
from setuptools import Extension, setup
from Cython.Build import cythonize

extensions = [
    Extension(
        # 这里写完整包名
        name='package1.package2.square',
        # 这里包含Cython文件和C++源文件
        sources=[
            'package1/package2/square.pyx',
            'package1/package2/_square.cpp',
        ],
        # 这里写编译flags；
        # - 写`-std=c++11`因为我们用了`auto`关键字
        # - 写`-DNDEBUG`是为了忽略所有`assert`（虽然这里并没有`assert`，只是为多举一个例子）
        extra_compile_args=['-std=c++11', '-DNDEBUG'],
        language='c++',
    ),
]

setup(
    # name参数可写可不写，这里没写
    #name='...',
    ext_modules=cythonize(extensions),
    zip_safe=False,
)
```

注意最后有一个`zip_safe=False`，根据[Building a Cython module using setuptools](http://docs.cython.org/en/latest/src/quickstart/build.html#building-a-cython-module-using-setuptools)，这是为避免一个导入错误：

> One caveat: the default action when running python setup.py install is to create a zipped egg file which will not work with cimport for pxd files when you try to use them from a dependent package. To prevent this, include zip\_safe=False in the arguments to setup().

最后我们来编译这个扩展模块。在命令行，项目根目录（即`setup.py`所在目录），执行：

```bash
python3 setup.py build_ext --inplace
```

为执行这条命令，Windows需要Visual Studio，Linux需要GNU工具链（`g++`），Mac需要XCode（`clang++`)。

为使用这个扩展模块，我们可以这样：

```python
from package1.package2.square import square

l1 = [1., 2., 3.]
print(square(l1))
```

输出

```
[1.0, 4.0, 9.0]
```

## 致谢

本文受[这个回答](https://stackoverflow.com/a/24836050/7881370)启发而创作。
