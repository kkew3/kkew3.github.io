---
layout: post
title:  "Effect of gamma in BN-VAE"
date:   2024-08-09 19:00:44 +0800
tags:   ml--bayes
---

## Abstract

This post discusses the effect of $\gamma$ in BN-VAE ([Zhu et al., 2020][zhu20]).

[zhu20]: https://arxiv.org/abs/2004.12585

## Introduction

BN-VAE (see more about it [here][bn-vae-jl] (in Chinese)) attempts to solve KL vanishing problem (a.k.a. posterior collapse) in Gaussian-VAE by batch-normalizing the variational posterior mean, which casts a positive lower bound on the [Kullback-Leibler divergence][kl] term (over the dataset) in [ELBO][elbo], thus avoiding KL vanishing problem.
The batch normalization procedure includes a fixed hyperparameter $\gamma \ge 0$, which controls the lower bound of the KL; the larger $\gamma$, the larger the lower bound.
When $\gamma=0$, KL vanishing occurs.

Zhu et al. (2020) visualizes the distribution of the variational posterior mean when $\gamma$ equals 0.3 and 1.
What will happen if $\gamma > 1$?
How does $\gamma > 0$ solves the KL vanishing problem?
We'll explore these questions below.

[bn-vae-jl]: https://kexue.fm/archives/7381

## $\gamma>1$ introduces posterior hole problem

Posterior hole problem happens when the aggregate variational posterior (a.k.a. average encoder distribution ([Hoffman & Johnson, 2016][hoffman16])) does not match the prior.
When measured in KL divergence, this means:

$$
D_{KL}(q_\phi(z) \parallel p(z)) > 0
$$

Here, $q_\phi(z) = \sum_{i=1}^N \frac{1}{N} q_\phi(z \mid x_i)$ where $N$ is the dataset size, is the aggregate variational posterior.

In Gaussian-VAE, the variational posterior $q_\phi(z \mid x_i) = \mathcal N(z \mid \mu_i, \sigma_i^2)$, where $(\mu_i,\sigma_i^2)$ are typically computed by a neural network called the inference network ([Kingma & Welling, 2013][vae]) parameterized by $\phi$ given $x_i$; and $q_\phi(z \mid x_i)$ can usually be factorized into each dimension $j$ as $q_\phi(z \mid x_i) = \prod_{j=1}^d q_\phi(z_j \mid x_i)$, where each $q_\phi(z_j \mid x_i)$ is an univariate Gaussian parameterized by $(\mu_{ij}, \sigma_{ij}^2)$.
Thus, the aggregate variational posterior is an $N$-mixture of Gaussians whose mean, at each dimension $j$, is $\bar\mu_j = \frac{1}{N}\sum_{i=1}^N \mu_{ij}$ and variance is $\bar\sigma_j^2 = \frac{1}{N}\sum_{i=1}^N \sigma_{ij}^2$.

If $q_\phi$ is transformed according to BN-VAE, then $\bar\mu_j = \beta$ where $\beta$ is a learnable parameter.
Furthermore, we have variance $\mathbb E_{q_\phi(z_j)}[z_j^2] - \mathbb E_{q_\phi(z_j)}[z_j]^2 = \gamma^2 + \bar\sigma^2$.
If we follow Zhu et al. (2020) to use a standard Gaussian $\mathcal N(z \mid \mathbf 0, \mathbf I)$ as prior $p$, then according to [this post][kllb-norm], $D_{KL}(q_\phi(z) \parallel p(z)$, at each dimension $j$, will be lower bounded by $D_{KL}(q_0(z_j) \parallel p(z_j))$ where $q_0(z_j) = \mathcal N(z_j \mid \beta, \gamma^2 + \bar\sigma^2)$, which is consistently greater than zero when $\gamma > 1$ ([Razavi et al., 2019][razavi19]).
It follows immediately ([Soch, Joram, et al., 2024][kl-add]), that $D_{KL}(q_\phi(z) \parallel p(z)) \ge \sum_{j=1}^d D_{KL}(q_0(z_i) \parallel p(z_i)) > 0$.

[kl]: https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence
[elbo]: https://en.wikipedia.org/wiki/Evidence_lower_bound
[hoffman16]: https://approximateinference.org/2016/accepted/HoffmanJohnson2016.pdf
[vae]: https://arxiv.org/pdf/1312.6114
[kllb-norm]: https://kkew3.github.io/2024/08/09/lower-bound-of-kl-divergence-between-any-density-and-gaussian.html
[razavi19]: https://arxiv.org/pdf/1901.03416
[kl-add]: https://statproofbook.github.io/P/kl-add


_TO BE CONTINUED_
