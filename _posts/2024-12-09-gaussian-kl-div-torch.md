---
layout: post
title:  "KL divergence between two full-rank Gaussians in PyTorch"
date:   2024-12-09 16:03:00 +0800
tags:   math--prob
---

## Abstract

In this post, we will go through the [PyTorch][torch] code to compute the Kullback-Leibler divergence between two full-rank Gaussians.
The code might be useful if one considers using full-rank Gaussian as variational posterior while training a [variational autoencoder][vae].

[torch]: https://pytorch.org/
[vae]: https://arxiv.org/abs/1312.6114

## KL divergence between two full-rank Gaussians

It's common practice to parameterize the covariance matrix $\boldsymbol\Sigma$ of a $d$-dimensional full-rank Gaussian using a $D$-dimensional vector of nonzero elements of $\mathbf L$, where $D = d(1+d)/2$ and $\boldsymbol\Sigma = \mathbf L \mathbf L^\top$ is the Cholesky decomposition.
So we will assume it here.
Note that the diagonal of $\mathbf L$ must be positive so that $\boldsymbol\Sigma$ is positive definite.
We will enforce this by taking the exponential on the diagonal elements (e.g. the first $d$ elements of our parameterization).

Let the two Gaussians be $p(\boldsymbol x) = \mathcal N(\boldsymbol x \mid \boldsymbol\mu_1, \boldsymbol\Sigma_1)$ and $q(\boldsymbol x) = \mathcal N(\boldsymbol x \mid \boldsymbol\mu_2, \boldsymbol\Sigma_2)$.
Per [The Book of Statistical Proofs][mvn-kl], the KL divergence between them is:

$$
D_\mathrm{KL}(p \parallel q) = \frac{1}{2}\left((\boldsymbol\mu_2 - \boldsymbol\mu_1)^\top \boldsymbol\Sigma_2^{-1} (\boldsymbol\mu_2 - \boldsymbol\mu_1) + \operatorname{tr}(\boldsymbol\Sigma_2^{-1} \boldsymbol\Sigma_1) - \log \frac{\det \boldsymbol\Sigma_1}{\det \boldsymbol\Sigma_2} - d\right)\,.
$$

Plugging in our parameterization of the covariance matrices:

$$
\begin{aligned}
    D_\mathrm{KL}(p \parallel q)
    &= \frac{1}{2}\left((\boldsymbol\mu_2 - \boldsymbol\mu_1)^\top \mathbf L_2^{-\top} \mathbf L_2^{-1} (\boldsymbol\mu_2 - \boldsymbol\mu_1) + \operatorname{tr}((\mathbf L_2 \mathbf L_2^\top)^{-1} (\mathbf L_1 \mathbf L_1^\top)) - \log \frac{\det(\mathbf L_1 \mathbf L_1^\top)}{\det(\mathbf L_2 \mathbf L_2^\top)} - d\right)\\
    &= \frac{1}{2}\left((\mathbf L_2^{-1} (\boldsymbol\mu_2 - \boldsymbol\mu_1))^\top (\mathbf L_2^{-1} (\boldsymbol\mu_2 - \boldsymbol\mu_1)) + \operatorname{tr}((\mathbf L_2^{-1} \mathbf L_1)^\top (\mathbf L_2^{-1} \mathbf L_1)) - 2\log\frac{\det\mathbf L_1}{\det\mathbf L_2} - d\right)\,.\\
\end{aligned}
$$

We have used the following facts:

- the [cyclic property of trace][trace-cyclic];
- $\det \mathbf A = \det \mathbf A^\top$;
- $\log\det (\mathbf A \mathbf B) = \log\det(\mathbf A) + \log\det(\mathbf B)$.

It follows that:

$$
D_\mathrm{KL}(p \parallel q) = \frac{1}{2}\big(\boldsymbol y^\top \boldsymbol y + \|\mathbf M\|_F^2 - 2 (\operatorname{tr}(\log \mathbf L_1) - \operatorname{tr}(\log \mathbf L_2)) - d\big)\,,
$$

where $\mathbf L_2 \boldsymbol y = \boldsymbol\mu_2 - \boldsymbol\mu_1$, and $\mathbf L_2 \mathbf M = \mathbf L_1$.

We have denoted:

- $\\|\cdot\\|_F$ as the Frobenius norm of a matrix;
- $\log \mathbf A$ as the elementwise logarithm of $\mathbf A$.

We have used the following facts:

- $\operatorname{tr}(\mathbf A^\top \mathbf A) = \\|\mathbf A\\|_F^2$;
- $\log\det \mathbf L = \operatorname{tr}(\log \mathbf L)$ when $\mathbf L$ is a lower triangular matrix.

[mvn-kl]: https://statproofbook.github.io/P/mvn-kl
[trace-cyclic]: https://en.wikipedia.org/wiki/Trace_(linear_algebra)#Cyclic_property

### Code

```python
import torch
from torch import distributions as D


def form_cholesky_tril_from_elements(d, scale_tril_elems):
    """
    Form the Cholesky lower triangular matrix from its elements.

    Args:
        d (int): The number of rows/columns in the square matrix.
        scale_tril_elems (torch.Tensor): The Cholesky lower triangular
            elements, of shape (batch_size, (1 + d) * d // 2).

    Returns:
        torch.Tensor: A tensor of shape (batch_size, d, d).
    """
    batch_size = scale_tril_elems.size(0)
    device = scale_tril_elems.device
    i, j = torch.tril_indices(d, d, device=device)
    l_mat = torch.zeros(batch_size, d, d, device=device)
    l_mat[:, i, j] = scale_tril_elems
    l_mat_diag = l_mat.diagonal(dim1=1, dim2=2)
    l_mat_diag.copy_(l_mat_diag.exp())
    return l_mat


d = 3
batch_size = 5


def groundtruth(mean1, scale_tril1, mean2, scale_tril2):
    p = D.MultivariateNormal(loc=mean1, scale_tril=scale_tril1)
    q = D.MultivariateNormal(loc=mean2, scale_tril=scale_tril2)
    return D.kl_divergence(p, q)


def ours(mean1, scale_tril1, mean2, scale_tril2):
    y = torch.linalg.solve_triangular(
        scale_tril2, (mean2 - mean1).unsqueeze(-1), upper=False)
        .squeeze(-1)
    y2 = y.square().sum(-1)
    M = torch.linalg.solve_triangular(scale_tril2, scale_tril1, upper=False)
    M2 = M.square().flatten(-2, -1).sum(-1)
    return 0.5 * (y2 + M2 - 2 * (
        scale_tril1.diagonal(dim1=-2, dim2=-1).log().sum(-1)
        - scale_tril2.diagonal(dim1=-2, dim2=-1).log().sum(-1)) - d)


# Randomize p and q's parameterization.
mean1 = torch.randn(batch_size, d)
mean2 = torch.randn(batch_size, d)
scale_tril1 = form_cholesky_tril_from_elements(
    d, torch.randn(batch_size, (1 + d) * d // 2))
scale_tril2 = form_cholesky_tril_from_elements(
    d, torch.randn(batch_size, (1 + d) * d // 2))

# Assert the correctness.
assert torch.allclose(groundtruth(mean1, scale_tril1, mean2, scale_tril2),
                      ours(mean1, scale_tril1, mean2, scale_tril2))
```

Profile our implementation:

`%timeit groundtruth(mean1, scale_tril1, mean2, scale_tril2)` (baseline):

```
164 μs ± 178 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)
```

`%timeit ours(mean1, scale_tril1, mean2, scale_tril2)` (our implementation):

```
46.2 μs ± 71.6 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)
```
