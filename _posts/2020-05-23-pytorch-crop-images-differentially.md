---
layout: post
title:  "PyTorch crop images differentially"
date:   2020-05-23 02:24:21 +0800
categories: pytorch math
---

# Intro

[PyTorch](https://pytorch.org) provides a variety of means to crop images. For example, [torchvision.transforms](https://pytorch.org/docs/stable/torchvision/transforms.html) provides several functions to crop `PIL` images; [PyTorch Forum](https://discuss.pytorch.org/t/how-to-crop-image-tensor-in-model/8409/3) provides an answer of how to crop image in a differentiable way (differentiable with respect to the image). However, sometimes we need a fully differentiable approach for the *cropping* action itself. How shall we implement that?

# Theory: Affine transformation

Before reaching the answer, we need first to learn about the image coordinate system in PyTorch. It is a left-handed Cartesian system origined at the middle of an image. The coordinate has been normalized to range $[-1,1]$, where $(-1,-1)$ indicates the top-left corner, and $(1,1)$ indicates the bottom-right corner, as pointed out by [the doc](https://pytorch.org/docs/stable/nn.functional.html#grid-sample).

Let $(x,y)$ be the top-left corner of the cropped image with respect to the coordinate of the original image; likewise, we denote $(x',y')$ as the bottom-right corner of the cropped image. It's clear that $(x,y)$ corresponds to $(-1,-1)$ with respect to the cropped image coordinate system, and $(x',y')$ corresponds to $(1,1)$. We'd like a function $f$ that maps from the cropped image system to the original image system for every point in the cropped image. Since only scaling and translation are involved, the function $f$ can be parameterized by an affine transformation matrix $\Theta$ such that

$$
\Theta =
\begin{pmatrix}
\theta_{11} & 0 & \theta_{13}\\
0 & \theta_{22} & \theta_{23}\\
0 & 0 & 1\\
\end{pmatrix}
$$

where $\theta_{12}=\theta_{21}=0$ since skewing is not involved. Denote $\mathbf{u}_H$ as the homogeneous coordinate of $\mathbf{u}=\begin{pmatrix}u & v\\ \end{pmatrix}^\intercal$ such that $\mathbf{u}_H=\begin{pmatrix}\mathbf{u}^\intercal&1\end{pmatrix}^\intercal$, $\Theta$ maps $\mathbf{u}_H$ with respect to the cropped image system to $\mathbf{x}_H$ with respect to the original image system, i.e. $\mathbf{x}_H = \Theta \mathbf{u}_H$. Thus,

$$
\begin{pmatrix}
x & x'\\
y & y'\\
1 & 1
\end{pmatrix} =
\begin{pmatrix}
\theta_{11} & 0 & \theta_{13}\\
0 & \theta_{22} & \theta_{23}\\
0 & 0 & 1\\
\end{pmatrix}
\begin{pmatrix}
-1 & 1\\
-1 & 1\\
1 & 1\\
\end{pmatrix}
$$

Solving the equations,

$$
\Theta =
\begin{pmatrix}
\frac{x'-x}{2} & 0 & \frac{x'+x}{2}\\
0 & \frac{y'-y}{2} & \frac{y'+y}{2}\\
0 & 0 & 1\\
\end{pmatrix}
$$

where $x'\ge x, y' \ge y$.

# Coding time

We'll need two functions:

1. [`torch.nn.functional.affine_grid`](https://pytorch.org/docs/stable/nn.functional.html#affine-grid) to convert the $\Theta$ parameterization to $f$
2. [`torch.nn.functional.grid_sample`](https://pytorch.org/docs/stable/nn.functional.html#grid-sample) to find the corresponding original image coordinate from each cropped image coordinate

```python
import torch
import torch.nn.functional as F

B, C, H, W = 16, 3, 224, 224  # batch size, input channels
                              # original image height and width
# Let `I` be our original image
I = torch.rand(B, C, H, W)
# Set the (x,y) and (x',y') to define the rectangular region to crop
x, y = -0.5, -0.3  # some examplary random coordinates;
x_, y_ = 0.7, 0.8  # in practice, (x,y,x_,y_) might be predicted
                   # as a tensor in the computation graph
# Set the affine parameters
theta = torch.tensor([
    [(x_-x)/2,       0, (x_+x)/2],
    [       0,(y_-y)/2, (y_+y)/2],
]).unsqueeze_(0).expand(B, -1, -1)
# compute the flow field;
# where size is the output size (scaling involved)
# `align_corners` option must be the same throughout the code
f = F.affine_grid(theta, size=(B, C, H//2, W//2), align_corners=False)
I_cropped = F.grid_sample(I, f, align_corners=False)
```

# Read also

- [https://discuss.pytorch.org/t/cropping-a-minibatch-of-images-each-image-a-bit-differently/12247](https://discuss.pytorch.org/t/cropping-a-minibatch-of-images-each-image-a-bit-differently/12247)
