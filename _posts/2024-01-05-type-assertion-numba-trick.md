---
layout: post
title:  "Assert variable types in numba"
date:   2024-01-05 17:04:32 +0800
tags:   dev--python
---

To assert that a variable is of any specific type, e.g., `float32[:]`, one may apply this trick that makes use of [`numba` signature](https://numba.pydata.org/numba-doc/latest/reference/types.html):

```python
import numba as nb


# Define an auxiliary function that admits only the type you
# want to assert, e.g. float32[:]
assert_f32_1d = nb.njit(nb.none(nb.float32[:]))(lambda x: None)

def function_to_debug_type(x, y, z):
    ...
    some_variable = ...
    ...
    # If `some_variable` is not of type float32[:], numba will
    # point it out.
    assert_f32_1d(some_variable)
```
