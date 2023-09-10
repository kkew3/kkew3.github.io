---
layout: post
title:  "Make use of openmp via cython on macOS"
date:   2023-09-10 16:49:00 +0800
tags:
- Cython
- openmp
---

## Abstract

This post gives a concise example on how to use [OpenMP](https://www.openmp.org) in [Cython](https://cython.readthedocs.io/en/latest/index.html) on macOS.

## Prerequisite

Install OpenMP.

```bash
brew install libomp
```

Install [numpy](https://numpy.org/) (used in the example) and Cython.

```bash
conda install numpy cython
```

My Cython version is `3.0.0`.

## Example

In `test.pyx`, we implement the [log-sum-exp trick](https://gregorygundersen.com/blog/2020/02/09/log-sum-exp/) in Cython.

```pyrex
from cython.parallel cimport prange
from libc.math cimport exp, log, fmax
cimport cython


@cython.boundscheck(False)
@cython.wraparound(False)
cdef double c_max(
    int N,
    double *a,
) nogil:
    cdef int i
    cdef double b = a[0]
    for i in range(1, N):
        b = fmax(b, a[i])
    return b


@cython.boundscheck(False)
@cython.wraparound(False)
cdef double c_logsumexp(
    int N,
    double *a,
) nogil:
    cdef int i
    cdef double b = c_max(N, a)
    cdef double x = 0.0
    for i in prange(N):
        x += exp(a[i] - b)
    x = b + log(x)
    return x


def logsumexp(double [::1] a):
    return c_logsumexp(a.shape[0], &a[0])
```

Note how to write the `setup.py`:

```python
from setuptools import Extension, setup
from Cython.Build import cythonize


extensions = [
    Extension(
        'test',
        sources=['test.pyx'],
        extra_compile_args=['-Xpreprocessor', '-fopenmp'],
        extra_link_args=['-lomp'],
    ),
]

setup(
    ext_modules=cythonize(extensions, language_level='3'),
    zip_safe=False,
)
```

The `-Xpreprocessor` is required for the openmp pragmas to be [processed](https://iscinumpy.gitlab.io/post/omp-on-high-sierra/).

## Build

```bash
python3 setup.py build_ext --inplace
```

After the build, `ls -F` output on my mac:

```
build/  setup.py  test.c  test.cpython-39-darwin.so*  test.pyx
```

## Test

```bash
python3 -m timeit -s 'from scipy.special import logsumexp; import numpy as np; a = np.random.randn(1000)' 'logsumexp(a)'
python3 -m timeit -s 'from test import logsumexp; import numpy as np; a = np.random.randn(1000)' 'logsumexp(a)'
```

The output:

```
10000 loops, best of 5: 32.1 usec per loop
50000 loops, best of 5: 6.66 usec per loop
```
