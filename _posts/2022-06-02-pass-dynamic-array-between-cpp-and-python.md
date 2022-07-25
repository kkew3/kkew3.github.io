---
layout: post
title:  "使用Cython在Python和C++间互传大小事先未知的numpy数组"
date:   2022-06-02 16:55:34 +0800
categories: python cpp Cython
---

# 从C++传到Python

常见的教程如[这个问题及回答](https://stackoverflow.com/q/17855032/7881370)是将大小已知的numpy数组传入传出C++，如确定会从C++传出大小为$M \times N$的矩阵。方法简单讲就是在Python端分配一个大小为$M \times N$的矩阵，把指向这个矩阵的指针传给C++，C++负责修改矩阵的内容，结束后矩阵就自动“传回”了。

然而有时我们事先不知道从C++传回的矩阵是多大，这时我们可以用[这个回答](https://python.tutorialink.com/passing-c-vector-to-numpy-through-cython-without-copying-and-taking-care-of-memory-management-automatically/)所提及的技术，即从C++传回`std::vector`，然后在Python端把它无拷贝地转成numpy数组。

例子：从C++传回$M \times 2$大小的矩阵，$M$在Python端未知。例子主要来源于网络，但我稍微换了一下应用，并修改了里面的谬误。

`doit.h`:

```cpp
#ifndef _DOIT_H_
#define _DOIT_H_
#include <vector>
std::vector<long> arange2d();
#endif
```

`doit.cpp`:

```cpp
#include "doit.h"
std::vector<long> arange2d() {
	std::vector<long> arr(10);
	long x = 0;
	for (auto i = arr.begin(); i != arr.end(); ++i) {
		*i = x++;
	}
	return arr;
}
```

`fast.pyx`:

```python
from libcpp.vector cimport vector

cdef extern from 'doit.h':
    vector[long] arange2d()

cdef class ArrayWrapper:
    cdef vector[long] v
    cdef Py_ssize_t shape[2];
    cdef Py_ssize_t strides[2];

    def set_data(self, vector[long]& data):
        self.v.swap(data)  # 注(1)

    def __getbuffer__(self, Py_buffer *buf, int flags):
        self.shape[0] = self.v.size() // 2
        self.shape[1] = 2
        self.strides[0] = self.shape[1] * sizeof(long)
        self.strides[1] = sizeof(long)

        # 注(2)
        buf.buf = <char *> self.v.data()
        buf.format = 'l'  # 注(3)
        buf.internal = NULL
        buf.itemsize = <Py_ssize_t> sizeof(long)
        buf.len = self.v.size() * sizeof(long)
        buf.ndim = 2
        buf.obj = self
        buf.readonly = 0
        buf.shape = self.shape
        buf.strides = self.strides
        buf.suboffsets = NULL

def pyarange2d():
    cdef vector[long] arr = arange2d()
    cdef ArrayWrapper wrapper = ArrayWrapper()
    wrapper.set_data(arr)
    return np.asarray(wrapper)
```

- 注(1)：`std::vector<T>::swap`完成了无拷贝传值，另一种方法是用`std::move`，不过那需要`cdef extern from '<utility>' namespace 'std' nogil: vector[long] move(vector[long])`，应该是这样，不过我没试过
- 注(2)：numpy的Buffer Protocol见[此处](https://docs.python.org/3/c-api/buffer.html#buffer-structure)，里面讲了`buf`需要设置哪些属性
- 注(3)：`buf.format`如何设置见[此处](https://docs.python.org/3/library/struct.html#format-characters)

至于从C++传回Python的多维数组有两个及以上的维度不知道的话（已知维度总数`ndim`），网络上没找到答案，但我是这么做的：

1. 传给C++一个指向`Py_ssize_t`类型、长度为`ndim`的数组（即待传回数组的`shape`）的指针
2. C++传回一个`std::vector`并修改`shape`元素为合适的值
3. 按照`shape`及`std::vector`的元素类型填写`buf`的属性，完成`std::vector`到numpy数组的转换

# 从Python传到C++

这应该已经耳熟能详了，我就不在此赘述了。不过有一点需要注意。传`double`数组时没问题，各平台`double`都对应`numpy.float64`。传`int`数组时需注意，Windows下对应`numpy.int32`、Linux/Mac下对应`numpy.int64`。所以直接用传`double`数组的方法传`int`数组会报这个错：

```
Cannot assign type 'int_t *' to 'int *'
```

见[这个问题](https://stackoverflow.com/q/72470641/7881370)（就是我提的）。目前我还没有优雅的解决方法。我笨拙的方法（受[ead](https://stackoverflow.com/users/5769463/ead)的启发）（请对照着“这个问题”看）如下：把所有的`int`全替换为`int64_t`（或`int32_t`，一致就行），例如`int * => int64_t *`、`np.int_t => np.int64_t`，然后在`dotit.h`包含头文件的地方加上`#include <cstdint>`，在`q.pyx`头部加上`from libc.stdint cimport int64_t`。应该就可以编译了。

补充一点我近期观察到的：以上workaround在Windows下（Visual Studio 2022）貌似不行，会报不能将`numpy`的`int32_t`转为`int32_t`，类似这样的错。在Darwin和Linux下都是能通过编译的。
