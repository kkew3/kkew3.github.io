---
layout: post
title:  "An attempt to build fully differentiable alternative of (non-negative) matching pursuit algorithm for solving L0-sparsity dictionary learning"
date:   2024-01-26 12:14:34 +0800
tags:   ml--dict
---

## Introduction

In sparse dictionary learning, sparse coding and dictionary update are solved in an alternating manner ([Aharon et al., 2006][1]).
In sparse coding stage, the following problem is solved given the dictionary $\mathbf D \in \mathbb R^{d \times n}$ and signals $y_j \in \mathbb R^d$:

$$
\min_{x_j}\ \|y_j-\mathbf D x_j\|_2^2 \quad \text{s.t. } \|x\|_0 \le K\,,\tag{1}
$$

where $K$ is the sparsity.
Sometimes, there's an additional constraint $x_j \succeq 0$ if non-negative sparse coding is required.
Since $L_0$ constraint is intractable to optimize exactly, either approximate greedy algorithm like (non-negative) orthogonal matching pursuit ([Cai & Wang, 2011][2]; [Yaghoobi et al., 2015][3]; [Nguyen et al., 2019][4]), or relaxation of $L_0$ to $L_1$ sparsity as (non-negative) basis pursuit ([Chen & Donoho, 1994][5]; [Gregor & LeCun, 2010][6]; [Zhang et al., 2018][7]; [Tolooshams & Ba, 2022][8]) are regarded as idiomatic solutions.

## Proposed method

([Louizos et al., 2018][9]) suggests a novel approach to handle the intractability of $L_0$ constraint.
Instead of tackling the $L_0$ constraint directly, the authors address the expectation of the $L_0$ norms by introducing Bernoulli random variables.
In the parlance of the sparse coding problem (1),

$$
\min_{x_j,\pi_j}\ \mathbb E_{q(z_j \mid \pi_j)}\left[\|y_j - \mathbf D (x_j' \odot z_j)\|_2^2\right] \quad \text{s.t. } \mathbf 1^\top \pi_j \le K\,,\tag{2}
$$

where $x_j$ has been reparameterized as $x_j' \odot z_j$, and for each $i$, $z_{ji} \sim \mathrm{Bernoulli}(\pi_{ji})$, $x_{ji}' \in \mathbb R$, the symbol $\odot$ denotes elementwise product.
Note that Equation (2.1) can be trivially extend to non-negative sparse coding case by reparameterization $x_j := \exp(x_j') \odot z_j$ or $x_j := \mathrm{softplus}(x_j') \odot z_j$, where $\mathrm{softplus}(\cdot) = \log(1 + \exp(\cdot))$.
([Louizos et al., 2018][9]) further introduces a smoother on the discrete random variable $z_j$ to allow for reparameterization trick ([Kingma & Welling, 2014][10]; [Rezende et al., 2014][11]), and the expectation in Equation (2) can be estimated by Monte Carlo sampling.

To solve the constrained minimization in Equation (2), it's natural to proceed using Lagrangian multiplier and optimize under bound constraint only:

$$
\min_{x_j,\pi_j}\max_{\lambda_j \ge 0}\ \mathbb E_{q(z_j \mid \pi_j)}\left[\|y_j - \mathbf D (x_j' \odot z_j)\|_2^2\right] + \lambda_j(\mathbf 1^\top \pi_j - K)\,.\tag{3}
$$

On the one hand, one may optimize $x_j,\pi_j,\lambda_j$ jointly via gradient descent.
However, it's worthy noting that one must perform gradient *ascent* on $\lambda_j$, which can be achieved by negating its gradient before the descent step.
On the other hand, [dual gradient ascent][12] can be adopted.
Here, given fixed $\lambda_j$, the objective (3) is minimized till a critical point; then given fixed $x_j$ and $\pi_j$, $\lambda_j$ is updated with one-step gradient ascent; finally, iterate.

In practice, potentially a great number of signals are required to be sparse coded given the dictionary:

$$
\min_{\boldsymbol x,\boldsymbol\pi}\max_{\boldsymbol\lambda \succeq 0}\ \sum_{j=1}^m \left\{\mathbb E_{q(z_j \mid \pi_j)}\left[\|y_j - \mathbf D (x_j' \odot z_j)\|_2^2\right] + \lambda_j(\mathbf 1^\top \pi_j - K)\right\}\,.\tag{4}
$$

It's not uncommon that all the variables to optimize, especially $\\{x_j,\pi_j\\}_{j=1}^m$, are unable to fit into memory, thus failing to run gradient descent.
Notice that for each $j$, the optimal solution $(x_j^\ast,\pi_j^\ast)$ are related to $(y_j,\lambda_j)$; that is, $x_j^\ast = x(y_j,\lambda_j)$, $\pi_j^\ast = \pi(y_j,\lambda_j)$.
Therefore, I propose to perform amortized inference: to use a neural network $f$ parameterized by $\boldsymbol\phi$ that takes as input $(y_j,\lambda_j)$ to predict $x_j$ and $\pi_j$.
I found the use of ReLU activation in such network promotes training the most.
The objective (4) now becomes:

$$
\min_{\boldsymbol\phi} \max_{\boldsymbol\lambda \succeq 0}\ \sum_{j=1}^m \left\{\mathbb E_{q(z_j \mid \boldsymbol\phi)} \left[\|y_j - \mathbf D (f_x(y_j,\lambda_j;\boldsymbol\phi) \odot z_j)\|_2^2\right] + \lambda_j (\mathbf 1^\top f_\pi(y_j,\lambda_j;\boldsymbol\phi) - K)\right\}\,.\tag{5}
$$

With dictionary learning, the dictionary need to be learned.
Using the objective (5), I found it preferable to optimize using the procedure below:

1. Given $\boldsymbol\lambda$, **reinitialize** $\boldsymbol\phi$, and jointly learn $\boldsymbol\phi$ and $\mathbf D$ until stationary point.
2. Given $\boldsymbol\phi$ and $\mathbf D$, perform one-step gradient ascent on $\boldsymbol\lambda$.
3. Iterate.

I found the reinitialization step on the amortized network critically important.
Without it, the network tends to predict all-zero and eventually learns nothing.
However, the dictionary needs to be initialized only at the very beginning.

## Experiments

For dictionary learning without non-negativity constraint on sparse coding, I compared against ([Rubinstein et al., 2008][13]) in image denoising.
My proposed fully differentiable solution converges slower and denoises poorer than K-SVD supported by batch OMP.

For dictionary learning *with* non-negative constraint on sparse coding, I compare against ([Nguyen et al., 2019][4]) in exploration of atoms of discourse, which is known to admit a non-negative sparse coding form ([Arora et al., 2018][14]).
While being faster, my proposed method still performs worse than non-negative OMP, in that the learned dictionary atoms are mostly not atoms of discourse.

Hence, this is the main reason why I record my attempt here in a post rather than write a paper.
Perhaps, the proposed method is promising, but it's not well-prepared yet.



[1]: https://www.khoury.northeastern.edu/home/eelhami/courses/EE290A/K-SVD_Elad.pdf
[2]: https://dspace.mit.edu/bitstream/handle/1721.1/72024/Wang_Orthogonal%20Matching.pdf
[3]: https://ieeexplore.ieee.org/abstract/document/7012095
[4]: https://hal.science/hal-02049424/file/paper1_hal.pdf
[5]: http://redwood.psych.cornell.edu/discussion/papers/chen_donoho_BP_intro.pdf
[6]: https://dl.acm.org/doi/abs/10.5555/3104322.3104374
[7]: https://mayhhu.github.io/ch/pdf/2018_L1-NNSO-Optim_ZHYW.pdf
[8]: https://arxiv.org/pdf/2106.00058
[9]: https://arxiv.org/pdf/1712.01312.pdf
[10]: https://arxiv.org/abs/1312.6114
[11]: http://proceedings.mlr.press/v32/rezende14.pdf
[12]: https://www.stat.cmu.edu/~ryantibs/convexopt-F18/lectures/dual-ascent.pdf
[13]: https://csaws.cs.technion.ac.il/~ronrubin/Publications/KSVD-OMP-v2.pdf
[14]: https://arxiv.org/abs/1601.03764
