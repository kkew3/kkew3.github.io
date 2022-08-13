---
layout: post
title:  "Notes on building Cython using setup.py"
date:   2022-08-09 16:24:19 +0800
categories: python Cython
---

# Basic structure

```python
from setuptools import Extension, setup
from Cython.Build import cythonize

extensions = [
    Extension(
        name=...,
        sources=[
            ...
        ],
        include_dirs=[
            ...
        ],
        library_dirs=[
            ...
        ],
        libraries=[
            ...
        ],
        runtime_library_dirs=[
            ...
        ],
        define_macros=[
            (..., ...),
            ...
        ],
        extra_compile_args=[
            ...
        ],
        extra_link_args=[
            ...
        ],
        language='...',
    ),
    ...
]

setup(
    ext_modules=cythonize(extensions, language_level='3'),
    zip_safe=False,
)
```

Notes:

- `name=...`: to be explained in detail below
- `sources=[...]`: from my experiments, seem must contain one and only one `.pyx` Cython source
- `language_level='3'` is used when developing in Python 3.
- `zip_safe=False` is used as per [cython doc](https://cython.readthedocs.io/en/latest/src/userguide/source_files_and_compilation.html#configuring-the-c-build)
- `define_macros=[("NPY_NO_DEPRECATED_API", "NPY_1_7_API_VERSION")]` can be used when devloping using newer version of `numpy`, to avoid compile-time warnings, despite harmless

## Name of Extension

> the full name of the extension, including any packages – ie. not a filename or pathname, but Python dotted name

For example, a name `foo.bar` will generate `./foo/bar.*.so` file, where `*` can be obtained by  invoke on command line `python3-config --extension-suffix`, e.g. `./foo/bar.cpython-39-darwin.so`.
The file path is relative to build root, the location where `setup.py` sits.

# Precedence of import

Suppose the extension we are talking about is named `foo.bar`.
Let's assume there's already a directory named `./foo/bar/`.
Open Python prompt under build root and type `from foo.bar import xxx` where `xxx` is anything defined in `foo.bar`.
This should work fine.
Now add an empty `foo/bar/__init__.py`.
Repeat the above process; it should echo `AttributeError` on `xxx`.
This means that the Python package `foo.bar` takes precedence over the extension module `foo.bar`.

Another circumstance.
Again the extension is named `foo.bar`.
However, there's now a directory `./foo/` with `bar.py` and `__init__.py` inside.
From my experiment, this time extension `foo.bar` takes precedence over the Python package `foo.bar`.

It appears quite involved to me.
So the best practice might be just to avoid having name collision with Python module/package.

# Useful links

- [`setuptools` Extension doc](https://setuptools.pypa.io/en/latest/userguide/ext_modules.html)
- [Cython source files and compilation](https://cython.readthedocs.io/en/latest/src/userguide/source_files_and_compilation.html)
- [Using C++ in Cython](https://cython.readthedocs.io/en/latest/src/userguide/wrapping_CPlusPlus.html)
