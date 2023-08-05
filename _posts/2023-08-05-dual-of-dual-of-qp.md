---
layout: post
title:  "The dual of the dual of a QP is itself"
date:   2023-08-05 18:54:12 +0800
categories: math
---

Given a Quadratic Program (QP), we will show that the dual of the dual of the QP is itself.

Let the QP be in its standard form:

$$
\min_{\boldsymbol x}\frac{1}{2}\boldsymbol x^\top\mathbf Q\boldsymbol x + \boldsymbol p^\top\boldsymbol x\,;
\quad\text{s.t. }\mathbf A\boldsymbol x=\boldsymbol b,\ x_i \ge 0\,,
$$

where $\mathbf Q \succ 0$ is positive definite.
The Lagrangian is

$$
L(\boldsymbol x,\boldsymbol\lambda,\boldsymbol\mu) = \frac{1}{2}\boldsymbol x^\top\mathbf Q\boldsymbol x + \boldsymbol p^\top\boldsymbol x + \boldsymbol\lambda^\top(\mathbf A\boldsymbol x-\boldsymbol b)-\boldsymbol\mu^\top\boldsymbol x\,,\tag{1}
$$

where $\mu_i \ge 0$.
Since $\mathbf Q \succ 0$, we may find the minimum of the $(1)$ with respect to $\boldsymbol x$ by driving $\partial L/\partial\boldsymbol x$ to $\mathbf 0$:

$$
\mathbf Q\boldsymbol x + p + \mathbf A^\top\boldsymbol\lambda - \boldsymbol\mu = \mathbf 0\,,
$$

and it follows that

$$
\boldsymbol x = \mathbf Q^{-1}(-\mathbf A^\top\boldsymbol\lambda - \boldsymbol p + \boldsymbol\mu)\,.
$$

Therefore, the dual formulation of the QP is:

$$
\max_{\boldsymbol\lambda,\boldsymbol\mu}-\frac{1}{2}(\mathbf A^\top\boldsymbol\lambda+\boldsymbol p-\boldsymbol\mu)^\top\mathbf Q^{-1}(\mathbf A^\top\boldsymbol\lambda+\boldsymbol p-\boldsymbol\mu)-\boldsymbol\lambda^\top\boldsymbol b\,;\quad\text{s.t. }\mu_i \ge 0\,.\tag{2}
$$

Now we will find the dual of the dual formulation.
First make Equation $(2)$ a minimization, and find its Lagrangian:

$$
\hat L(\boldsymbol\lambda,\boldsymbol\mu,\boldsymbol y) = \frac{1}{2}(\mathbf A^\top\boldsymbol\lambda+\boldsymbol p-\boldsymbol\mu)^\top\mathbf Q^{-1}(\mathbf A^\top\boldsymbol\lambda+\boldsymbol p-\boldsymbol\mu) + \boldsymbol\lambda^\top\boldsymbol b - \boldsymbol y^\top\boldsymbol\mu\,,\tag{3}
$$

where $y_i \ge 0$.
Since $\mathbf Q^{-1} \succ 0$, we may also find its minimum with respect to $\boldsymbol\lambda$ and $\boldsymbol\mu$ by driving corresponding partial derivatives to zero:

$$
\begin{align}
\frac{\partial\hat L}{\partial\boldsymbol\lambda} &= \frac{1}{2}(2\mathbf A\mathbf Q^{-1}\mathbf A^\top\boldsymbol\lambda+\mathbf A\mathbf Q^{-1}\boldsymbol p-\mathbf A\mathbf Q^{-1}\boldsymbol\mu+\mathbf A\mathbf Q^{-\top}\boldsymbol p-\mathbf A\mathbf Q^{-\top}\boldsymbol\mu)+\boldsymbol b = 0\,,\tag{4.1}\\
\frac{\partial\hat L}{\partial\boldsymbol\mu} &= \frac{1}{2}(-\mathbf Q^{-\top}\mathbf A^\top\boldsymbol\lambda-\mathbf Q^{-\top}\boldsymbol p-\mathbf Q^{-1}\mathbf A^\top\boldsymbol\lambda-\mathbf Q^{-1}\boldsymbol p+2\mathbf Q^{-1}\boldsymbol\mu) - \boldsymbol y = 0\,,\tag{4.2}\\
\end{align}
$$

where $\mathbf Q^{-\top} \equiv (\mathbf Q^{-1})^\top$.
Left-multiplying $(4.2)$ by $\mathbf A$ and adding it to $(4.1)$ yields

$$
\mathbf A\boldsymbol y=\boldsymbol b\,.\tag{5}
$$

This holds since positive definite matrices are symmetric.
It follows from $(4.2)$ that

$$
-\mathbf Q\boldsymbol y = \mathbf A^\top\boldsymbol\lambda-\boldsymbol\mu+\boldsymbol p\,.\tag{6.1}
$$

or

$$
\boldsymbol y = -\mathbf Q^{-1}(\mathbf A^\top\boldsymbol\lambda-\boldsymbol\mu+\boldsymbol p\,.\tag{6.2}
$$

Plugging $(6.1)$ and $(6.2)$ back to $(3)$ gives

$$
\begin{aligned}
\hat L(\boldsymbol\lambda,\boldsymbol\mu,\boldsymbol y)
&= \frac{1}{2}(\mathbf A^\top\boldsymbol\lambda+\boldsymbol p-\boldsymbol\mu)^\top\mathbf Q^{-1}\mathbf A^\top\boldsymbol\lambda+\boldsymbol p-\boldsymbol\mu - (\mathbf A^\top\boldsymbol\lambda-\boldsymbol\mu)^\top\mathbf Q^{-1}(\mathbf A^\top\boldsymbol\lambda+\boldsymbol p-\boldsymbol\mu)\\
&= \frac{1}{2}(-\mathbf Q\boldsymbol y)^\top\mathbf Q^{-1}(-\mathbf Q\boldsymbol y)-(-\mathbf Q\boldsymbol y-\boldsymbol p)^\top\mathbf Q^{-1}(-\mathbf Q\boldsymbol y)\\
&= -\frac{1}{2}\boldsymbol y^\top\mathbf Q\boldsymbol y-\boldsymbol p^\top\boldsymbol y\,.\\
\end{aligned}
$$

Together with Equation $(5)$ and $y_i \ge 0$, we have the dual of the dual formulation:

$$
\max_{\boldsymbol y}-\frac{1}{2}\boldsymbol y^\top\mathbf Q\boldsymbol y-\boldsymbol p^\top\boldsymbol y\,;\quad\text{s.t. }\mathbf A\boldsymbol y=\boldsymbol b,\ y_i \ge 0\,.
$$

Clearly, this is equivalent to the original QP.
