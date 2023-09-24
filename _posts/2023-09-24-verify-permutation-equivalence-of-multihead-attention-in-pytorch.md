---
layout: post
title:  "Verify permutation equivalence of Multi-Head Attention in PyTorch"
date:   2023-09-24 16:54:32 +0800
tags:   pytorch
---

It's well known that [Multi-Head Attention](https://arxiv.org/abs/1706.03762) is permutation equivalent (e.g. [here](https://uvadlc-notebooks.readthedocs.io/en/latest/tutorial_notebooks/tutorial6/Transformers_and_MHAttention.html)).
Let's verify it in PyTorch.

```python
import torch
from torch import nn

batch_size = 16
seq_length = 10
embed_dim = 384
n_heads = 8

attn = nn.MultiheadAttention(embed_dim, n_heads, batch_first=True)
X = torch.rand(batch_size, seq_length, embed_dim)
o = torch.randperm(seq_length)
z1, _ = attn(X, X, X)
z2, _ = attn(X[:, o], X[:, o], X[:, o])
print(torch.allclose(z1[:, o], z2))
```

Almost certainly, it will print a `False`.
What's going wrong?
It turns out that PyTorch uses `torch.float32` by default.
Let's increase the precision to `torch.float64`:

```python
import torch
from torch import nn

batch_size = 16
seq_length = 10
embed_dim = 384
n_heads = 8

attn = nn.MultiheadAttention(embed_dim, n_heads, batch_first=True).to(torch.float64)
X = torch.rand(batch_size, seq_length, embed_dim, dtype=torch.float64)
o = torch.randperm(seq_length)
z1, _ = attn(X, X, X)
z2, _ = attn(X[:, o], X[:, o], X[:, o])
print(torch.allclose(z1[:, o], z2))
```

It should print `True` now.
