---
layout: post
title:  "Variational Autoencoder training trick"
date:   2022-08-31 19:48:35 +0800
tags:   ml
---

A decent tutorial on Variational Autoencoder (VAE) can be found at [arXiv](https://arxiv.org/abs/1606.05908).
While I was playing with VAE, and trying the Gaussian output distribution with gradually higher dimensionality, I found a trick that ensure numerical stability at the beginning of training.
As we know, the "encoder" of VAE outputs $\mu_X$ and $\log\sigma_X^2$ given input $x$, and a $z$ is sampled from the Gaussian determined by $\mu_X$ and $\sigma_X^2$ afterwards.
To compute $\sigma_X^2$, $\sigma_X^2=e^{\log\sigma_X^2}$.

A problem arises, that $\log\sigma_X^2$ goes large enough such that $\sigma_X^2$ becomes floating-point infinity, especially when the mean and log variance is predicted by a dense linear layer and when the input dimension is high.
This is because, despite the fact that log variance is typically small at the end of training, it's value at the beginning of training is determined by random initialization of the dense linear layer.
Suppose that the linear layer is initialized as standard Gaussian.
With $K$ input neurons, each output element of the linear layer follows the distribution of the sum of $K$ standard Gaussian, whose variance is positively proportional to $K$.
It follows that the maximum of all output elements is proportional to the variance.
Therefore, there should exist an element in $\sigma_X^2$ that is $e^K$ times the expected range.
Naturally, when $K$ is large, it goes to floating-point infinity.

To solve the problem, we may goes one step further.
Rather than predict $\log\sigma^2$, we predict $K\log\sigma_X^2$, and in turn the output variance becomes $e^{(K\log\sigma_X^2)/K}$, which won't ever reach infinity.
Since $K$ is a constant throughout training, it won't cause any effect to the training overall.
