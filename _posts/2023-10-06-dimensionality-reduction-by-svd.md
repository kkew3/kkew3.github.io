---
layout: post
title:  "Dimensionality reduction by SVD"
date:   2023-10-06 16:43:30 +0800
tags:   math
---

Let $\mathbf X \in \mathbb R^{m \times d}$ be data matrix where the $t$-th row is the $t$-th item in the dataset.
One may achieve dimensionality reduction from $\mathbf X$ to $\tilde{\mathbf X}$, by computing the SVD: $\mathbf X = \mathbf U \mathbf S \mathbf V^\top$, and let $\tilde{\mathbf X} = \mathbf U_k\mathbf S_k$, where $\mathbf U_k$ is the first $k$ columns of $\mathbf U$ and $\mathbf S_k$ is a diagonal matrix of the first $k$ diagonal elements of $\mathbf S$.
The idea behind this process can be viewed as either the classic MDS or PCA.

In classic MDS, one wants to maintain as much as possible the inner product matrix $\mathbf X\mathbf X^\top = \sum_{j=1}^r \sigma_j^2 \boldsymbol u_j \boldsymbol u_j^\top$ where $\sigma_j$'s have been sorted in descending order.
Clearly, one may perform low-rank approximation of $\mathbf X$ by $\tilde{\mathbf X} = \mathbf U_k \mathbf S_k$ such that $\mathbf X \mathbf X^\top \approx \tilde{\mathbf X}\tilde{\mathbf X}^\top$.

In PCA, one aims to find the orthonormal transformation matrix $\mathbf V_k$, which is the first $k$ columns of the eigenvectors of the covariance matrix $\mathbf X^\top \mathbf X$ (up to a constant) where the eigenvalues have been sorted in descending order, and then reaches the low-dimensional representation $\mathbf X \mathbf V_k$, which is identical to $\mathbf U_k \mathbf S_k$.

One point to note is that, if the data matrix is arranged as $\mathbf X' \in \mathbb R^{d \times m}$, where each column is a vector in the dataset, and let $\mathbf X' = \mathbf U \mathbf S \mathbf V^\top$ instead, then the low-dimensional representation will be $\mathbf S_k \mathbf V_k^\top$.
Let's derive this with PCA:
The covariance matrix is now $\mathbf X' \mathbf X^{\prime\top}$ and so the transformed data matrix is $\mathbf U^\top \mathbf X'$.
By SVD, clearly it equals $\mathbf S_k V_k^\top$.

As a matter of fact, if we denote $\mathbf X=\mathbf U_1 \mathbf S_1 \mathbf V_1^\top$ and $\mathbf X^\top = \mathbf V_2 \mathbf S_2 \mathbf U_2^\top$, then it should turn out that $(\mathbf U_1 \mathbf S_1)^\top = \mathbf S_2 \mathbf U_2^\top$.
Let's write Python3 code to verify this:

```python
import numpy as np

X = np.random.randn(1000, 100)

U1, S1, VT1 = np.linalg.svd(X, full_matrices=False)
V2, S2, UT2 = np.linalg.svd(X.T, full_matrices=False)
assert np.allclose((U1 @ np.diag(S1)).T, np.diag(S2) @ UT2)
```

It could occurs that the assertion fails.
The reason is that given diagonal matrix $\mathbf Q$ where its diagonal elements be either $1$ or $-1$, and given SVD $\mathbf X = \mathbf U \mathbf S \mathbf V^\top$, the following holds for any such $\mathbf Q$: $\mathbf X = \mathbf U \mathbf Q \mathbf S \mathbf Q^\top \mathbf V^\top$.
We need to take into account this case.
Rewriting the code as:

```python
import numpy as np

X = np.random.randn(1000, 100)

U1, S1, VT1 = np.linalg.svd(X, full_matrices=False)
V2, S2, UT2 = np.linalg.svd(X.T, full_matrices=False)
# assert np.allclose((U1 @ np.diag(S1)).T, np.diag(S2) @ UT2)

q = np.mean(U1 / UT2.T, axis=0)
assert np.allclose(q, U1 / UT2.T)
Q = np.diag(q)
assert np.allclose((U1 @ np.diag(S1)).T, np.diag(S2) @ Q @ UT2)
```

We should now pass the assertion.
