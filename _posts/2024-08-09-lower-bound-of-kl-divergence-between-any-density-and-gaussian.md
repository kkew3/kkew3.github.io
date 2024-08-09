---
layout: post
title:  "Lower bound of KL divergence between any density and Gaussian"
date:   2024-08-09 17:03:39 +0800
tags:   math--prob
---

## Abstract

In this post, I explain how to derive a lower bound of the [Kullback-Leibler divergence][kl] between any density $q$, e.g. a Gaussian mixture, and a Gaussian $p$.

[kl]: https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence

## Framework

We may cast the problem finding the lower bound to an constrained minimization problem:

<div id="eq-1"></div>

$$
\begin{aligned}
    \min_{q'}\ &D_{KL}(q' \parallel p)\\
    \text{s.t. } &\int_{\mathcal X} q'(x)\,\mathrm dx = 1\\
                 &\ldots \ \text{other constraints}
\end{aligned}\tag{1}
$$

where $\mathcal X$ is the support of $q'$, and we'll fill in "other constraints" with what we know about the density $q$, like its mean and variance.
The solution of Equation (<a href="#eq-1">1</a>) will be the lower bound we're seeking for.

The [Lagrangian][lm] would be:

$$
L = \int_{\mathcal X} q'(x)\log \frac{q'(x)}{p(x)}\,\mathrm dx + \lambda_0 (\int_{\mathcal X} q'(x)\,\mathrm dx - 1) + \ldots \tag{2}
$$

Taking the functional derivative of $L$ with respect to $q'$ and letting it equal zero yields:

$$
\begin{aligned}
    0 &= 1 + \log q'(x) - \log p(x) + \lambda_0 + \ldots\\
    \log q'(x) &= -\lambda_0 - 1 + \log p(x) + \ldots\\
    q'(x) &= \exp(-\lambda_0 -1 + \log p(x) + \ldots)
\end{aligned}
$$

Finally, plugging $q'(x)$ back into the constraints and solve for the Lagrange multipliers $\lambda_0$, etc.

[lm]: https://en.wikipedia.org/wiki/Lagrange_multiplier

## Example

In this simple example, we assume that $p(x) = \mathcal N(x \mid 0, 1)$ be a standard univariate Gaussian, and assume that $q$ and $p$ have the same support.
Suppose also that we know the mean and variance of $q$ to be: $\mathbb E_q[x] = 0$, $\mathbb E_q[x^2] - \mathbb E_q[x]^2 = \mathbb E_q[x^2] = \sigma^2$.

The Lagrangian is:

<div id="eq-3"></div>

$$
\require{enclose}
L = \int_{-\infty}^\infty q'(x) \log \frac{q'(x)}{p(x)}\,\mathrm dx + \lambda_0 (\underbrace{\int_{-\infty}^\infty q'(x)\,\mathrm dx - 1}_{\substack{\enclose{circle}{1}}}) + \lambda_1 (\underbrace{\int_{-\infty}^\infty x^2 q'(x)\,\mathrm dx - \sigma^2}_{\substack{\enclose{circle}{2}}})\tag{3}
$$

where we have encoded the mean and variance constraints into one term (see why [here][maxent]).
Taking the derivative and letting it equal zero yields:

<div id="eq-4"></div>

$$
\begin{align}
    0 &= 1 + \log q'(x) - \log p(x) + \lambda_0 + \lambda_1 x^2\\
    \log q'(x) &\stackrel{1}{=} -\lambda_0 - 1 - (\frac{1}{2} + \lambda_1) x^2\\
    q'(x) &= \exp(-\lambda_0 - 1 - (\frac{1}{2} + \lambda_1) x^2)\tag{4}\\
\end{align}
$$

where equal sign '$1$' is because $\log p(x) = -\frac{1}{2}x^2 + C$, and the constant $C$ has been absorbed into $\lambda_0$.

Plugging Equation (<a href="#eq-4">4</a>) back to <a href="#eq-3">⓵</a> and solving the integral yields:

<div id="eq-5.1"></div>

$$
\frac{\sqrt{\pi}\exp(-\lambda_0 - 1)}{\sqrt{\frac{1}{2} + \lambda_1}} = 1\tag{5.1}
$$

Likewise, plugging (<a href="#eq-4">4</a>) back to <a href="#eq-3">⓶</a> and solving the integral yields:

<div id="eq-5.2"></div>

$$
\frac{\sqrt{\pi} \exp(-\lambda_0 - 1)}{2\sqrt{(\frac{1}{2} + \lambda_1)^3}} = \sigma^2\tag{5.2}
$$

Solving Equations (<a href="#eq-5.1">5.1</a>, <a href="#eq-5.2">5.2</a>) gives:

<div id="eq-6"></div>

$$
\begin{cases}
    \lambda_0 = -1 + \frac{1}{2} \log 2\pi\sigma^2\\
    \lambda_1 = -\frac{1}{2} + \frac{1}{2\sigma^2}\\
\end{cases}\tag{6}
$$

Plugging Equation (<a href="#eq-6">6</a>) to Equation (<a href="#eq-4">4</a>), it's immediate that

$$
q'(x) = \frac{1}{\sqrt{2\pi\sigma^2}}\exp(-\frac{x^2}{2\sigma^2})
$$

i.e., a Gaussian $\mathcal N(x \mid 0, \sigma^2)$.
Therefore, according to [this chapter][kl-norm],

$$
D_{KL}(q \parallel p) \ge \frac{1}{2}(\sigma^2 - \log\sigma^2 - 1)
$$



[maxent]: https://michael-franke.github.io/intro-data-analysis/the-maximum-entropy-principle.html#example-2-derivation-of-maximum-entropy-pdf-with-given-mean-mu-and-variance-sigma2
[kl-norm]: https://statproofbook.github.io/P/norm-kl
