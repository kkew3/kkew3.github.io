---
layout: post
title:  "A simple numerical method to compute matrix inversion"
date:   2024-02-26 18:57:01 +0800
tags:   math--la
---

I need to do matrix inversion in C recently; so I did some research on how to implement it.
While the requirement later proves unnecessary, I want to jot down my efforts on this subject for future reference.

([Pan & Schreiber, 1992][1]) proposed CUINV algorithm based on [Newton's iteration][2].
It's fast and simple to implement.
Here's my verbatim reimplementation in Python, which is simple(?) (see TODO in comment) to translate to C.

```python
import numpy as np

def cuinv(A, maxiter, tol):
    n = A.shape[0]
    I = np.eye(n)
    s = np.linalg.svd(A, compute_uv=False)  # TODO: how to implement this?
    a0 = 2 / (np.min(s)**2 + np.max(s)**2)
    X = a0 * A.T
    X_prev = np.copy(X)
    T = X @ A
    T2 = None
    t2_valid = False
    diff = tol + 1  # so that it runs at least one iteration

    for _ in range(maxiter):
        if diff < tol:
            break
        X = (2 * I - T) @ X
        if t2_valid:
            T = 2 * T - T2
        else:
            T = X @ A
        t2_valid = False
        if np.trace(T) < n - 0.5:
            T2 = T @ T
            delta = np.linalg.norm(T - T2, ord='fro')
            if delta >= 0.25:
                t2_valid = True
            else:
                rho = 0.5 - np.sqrt(0.25 - delta)
                X = 1 / rho * (T2 - (2 + rho) * T + (1 + 2 * rho) * I) @ X
                T = X @ A
        diff = np.linalg.norm(X - X_prev, ord='fro')
        X_prev = X
    return X
```


[1]: https://ntrs.nasa.gov/api/citations/19920002505/downloads/19920002505.pdf
[2]: https://aalexan3.math.ncsu.edu/articles/mat-inv-rep.pdf
