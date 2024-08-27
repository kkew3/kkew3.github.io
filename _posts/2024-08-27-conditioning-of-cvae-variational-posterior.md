---
layout: post
title:  "Conditioning of the variational posterior in CVAE (Sohn, 2015)"
date:   2024-08-27 19:03:07 +0800
tags:   math--prob ml--bayes
---

## Abstract

This post explores alternative conditioning of the variational posterior $q$ in CVAE ([Sohn et al. 2015][sohn15]), and concludes that the conditioning of $q$ on $y$ is important to predictive inference.

[sohn15]: https://papers.nips.cc/paper_files/paper/2015/hash/8d55a249e6baa5c06772297520da2051-Abstract.html

## Introduction

CVAE models the likelihood $p(y \mid x)$ as a continuous mixture of latent $z$:

<div id="eq-def"></div>

$$
p(y \mid x) = \int p_\theta(z \mid x) p_\theta(y \mid x,z)\,\mathrm dz\,. \tag{1}
$$

Since (<a href="#eq-def">1</a>) is intractable, Sohn et al. instead optimize its evidence lower bound (ELBO):

$$
\mathcal L_{\text{CVAE}}(x,y;\theta,\phi) = \mathbb E_q[\log p_\theta(y \mid x,z) - \log q_\phi(z \mid x,y)]\,. \tag{2}
$$

Here, the variational posterior $q$ conditions on $x$ and $y$.
At test time, the authors propose to use importance sampling leveraging the trained variational posterior:

<div id="eq-is"></div>

$$
p(y \mid x) \approx \frac{1}{S} \sum_{s=1}^S \frac{p_\theta(y \mid x,z_s) p_\theta(z_s \mid x)}{q_\phi(z_s \mid x,y)}\,, \tag{3}
$$

where $z_s \sim q_\phi(z \mid x,y)$.

What if $q$ conditions on $x$ only?
This post explores this possibility, and reaches the conclusion that without conditioning on $y$, $q$ at optimum won't ever attain the true posterior $p(z \mid x,y)$, and should not be otherwise better in terms of reducing the variance in importance sampling.

## Warm up: proving the effecacy of importance sampling

We assume that infinite data is available for learning, and $q$ is from a flexible enough probability family.
The data are drawn from the joint data distribution $p_D(x,y)$, where we have stressed with a subscript $D$.
We assume that $x$ is continuous and $y$ is discrete.
The goal is to maximize the expected ELBO in terms of $p_D(x,y)$.
However, we assume that $p_\theta(y \mid x,z)$ won't approaches to $p_D(y \mid x)$ whatever value $\theta$ picks.
We will drop $\theta$ and $\phi$ below for brevity.

We may easily pose this setup as a constrained maximization problem:
$\max \mathbb E[\log p(y,z \mid x) - \log q(z \mid x,y)]$ subject to $q$ being a probability, where the expectation is taken with respect to $p_D(x,y) q(z \mid x,y)$.

The Lagrangian is:

<div id="eq-lagrangian-q-xy"></div>

$$
\int \sum_y p_D(x,y) \int q(z \mid x,y) \log \frac{p(y,z \mid x)}{q(z \mid x,y)}\,\mathrm dz\,\mathrm dx + \int \sum_y \mu(x,y) \left(\int q(z \mid x,y)\,\mathrm dz - 1\right)\,\mathrm dx\,, \tag{4}
$$

where $\mu(x,y)$ is the Lagrange multiplier.
Now find the [Gateaux derivative][gateaux] and let it equal zero:

$$
0 = p_D(x,y) (\log p(y,z \mid x) - (1 + \log q(z \mid x,y)) + \mu(x,y))\,.
$$

Absorbing $p_D(x,y) > 0$ and the constant 1 into $\mu(x,y)$ yields:

$$
\log q(z \mid x,y) = \mu(x,y) + \log p(y,z \mid x)\,,
$$

where $\mu(x,y) = -\log \int p(y,z \mid x)\,\mathrm dz = -\log p_D(y \mid x)$.
It thus follows that, at optimum, $q(z \mid x,y) = p(z \mid x,y)$.
Hence, when evaluating Equation (<a href="#eq-is">3</a>), at optimum, the right hand side equals the left hand side with zero variance.

[gateaux]: https://www.youtube.com/watch?v=6VvmMkAx5Jc&t=982s

## Conditioning only on x gives worse approximation

Following the same setup as the previous section, we start from the Lagrangian (<a href="#eq-lagrangian-q-xy">4</a>).
Note that now we assume $q \triangleq q(z \mid x)$, and that the Lagrange multiplier is $\mu(x)$ instead of $\mu(x,y)$.
Rearranging the terms:

$$
\begin{multline}
    \int p_D(x) \int q(z \mid x) \left(\sum_y p_D(y \mid x) \log p(y,z \mid x) - \log q(z \mid x)\right)\,\mathrm dz\,\mathrm dz \\
    + \int \mu(x) \left(\int q(z \mid x)\,\mathrm dz - 1\right)\,\mathrm dx\,.
\end{multline}
$$

Let its Gateaux derivative with respect to $q$ equal zero:

$$
0 = p_D(x) \left(\sum_y p_D(y \mid x) \log p(y,z \mid x) - (1 + \log q(z \mid x))\right) + \mu(x)\,.
$$

Absorbing $p_D(x) > 0$ and the constant 1 into $\mu(x)$ yields:

$$
\log q(z \mid x) = \mu(x) + \sum_y p_D(y \mid x) \log p(z \mid x,y) - \mathbb H(p_D(y \mid x))\,,
$$

where $\mathbb H(p_D(y \mid x)) = -\sum_y p_D(y \mid x) \log p_D(y \mid x)$ is the entropy.
We see immediately that:

<div id="eq-main-result"></div>

$$
q(z \mid x) \propto \exp(\mathbb E_{p_D(y \mid x)}[\log p(z \mid x,y)])\,. \tag{5}
$$

This means that when not conditioning on $y$, $q(z \mid x)$ can never achieve the true posterior $p(z \mid x,y)$, unless $\mathbb H(p_D(y \mid x)) = 0$, which is unlikely to occur.
