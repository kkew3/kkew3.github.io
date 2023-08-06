---
layout: post
title:  "How to compute the intercept of C-SVM in primal and dual formulations"
date:   2023-08-05 16:08:26 +0800
categories: math ML
---

# Compute intercept in primal formulation

The primal SVM problem is:

$$
\min_{\boldsymbol w,b,\boldsymbol\xi} \frac{1}{2}\boldsymbol w^\top\boldsymbol w+C\sum_{i=1}^m\xi_i;\quad\text{s.t. }\ y_i f(\boldsymbol x_i) \ge 1-\xi_i,\ \xi_i \ge 0 \,,\tag{1}
$$

where the decision function $f(\boldsymbol x) \equiv \boldsymbol w^\top\phi(\boldsymbol x) + b$.
The Lagrangian is:

$$
L(\boldsymbol w,b,\boldsymbol\xi,\boldsymbol\alpha,\boldsymbol\mu) = \frac{1}{2}\boldsymbol w^\top\boldsymbol w + C\sum_{i=1}^m\xi_i + \sum_{i=1}^m\alpha_i\big(1-\xi_i-y_i f(\boldsymbol x_i)\big) - \sum_{i=1}^m\mu_i\xi_i\,,
$$

where $\alpha_i \ge 0$, $\mu_i \ge 0$.
The Karush-Kuhn-Tucker (KKT) conditions are:

$$
\begin{cases}
\boldsymbol w=\sum_{i=1}^m\alpha_i y_i \phi(\boldsymbol x_i) &\text{(stationarity)}\\
0=\sum_{i=1}^m\alpha_i y_i &\text{(stationarity)}\\
C=\alpha_i+\mu_i &\text{(stationarity)}\\
0=\alpha_i(y_i f(\boldsymbol x_i)-1+\xi_i) &\text{(complementary)}\\
0=\mu_i\xi_i &\text{(complementary)}\\
y_i f(\boldsymbol x_i)-1+\xi_i \ge 0 &\text{(primal feasibility)}\\
\xi_i \ge 0 &\text{(primal feasibility)}\\
\alpha_i \ge 0 &\text{(dual feasibility)}\\
\mu_i \ge 0 &\text{(dual feasibility)}\\
\end{cases}\,.
$$

Thus, we have

$$
\begin{cases}
y_i f(\boldsymbol x_i) \ge 1 &(\alpha_i=0)\\
y_i f(\boldsymbol x_i) \le 1 &(\alpha_i=C)\\
y_i f(\boldsymbol x_i) = 1 &(\text{otherwise})\\
\end{cases}\,.\tag{2}
$$

When $S=\\{j \mid 0 < \alpha_j < C\\} \neq \varnothing$, for each such $j$,

$$
\begin{aligned}
y_j (\boldsymbol w^\top\phi(\boldsymbol x_j)+b) &= 1\\
b &= y_j - \boldsymbol w^\top\phi(\boldsymbol x_j)\,;\\
\end{aligned}
$$

The second equality holds since $y_j = \pm 1$.
For numerical stability, we take the mean of all $b$'s as the final value of the intercept:

$$
b = \frac{1}{\\|S\\|}\sum_{j \in S} (y_j-\boldsymbol w^\top\phi(\boldsymbol x_j))\,.
$$

When $S=\varnothing$, taking the first two cases of Equation $(2)$, it follows that

$$
\begin{cases}
f(\boldsymbol x_i) \ge 1 &(\alpha_i=0,y_i=1)\\
f(\boldsymbol x_i) \le -1 &(\alpha_i=0,y_i=-1)\\
f(\boldsymbol x_i) \le 1 &(\alpha_i=C,y_i=1)\\
f(\boldsymbol x_i) \ge -1 &(\alpha_i=C,y_i=-1)\\
\end{cases}\,.
$$

Equivalently, we have

$$
\max_{j \in T_1}\{y_j - \boldsymbol w^\top\phi(\boldsymbol x_j)\} \le b \le \min_{j \in T_2}\{y_j - \boldsymbol w^\top\phi(\boldsymbol x_j)\}\,,
$$

where

$$
\begin{cases}
T_1 = \{j \mid \alpha_j=0,y_j=1\text{ or }\alpha_j=C,y_j=-1\}\\
T_2 = \{j \mid \alpha_j=0,y_j=-1\text{ or }\alpha_j=C,y_j=1\}\\
\end{cases}\,,
$$

The intercept is taken as the mean of the lower and upper bounds.

To compute $\boldsymbol w^\top\phi(\boldsymbol x)$ in above equations, simply plug in $\boldsymbol w=\sum_{i=1}^m\alpha_i y_i \phi(\boldsymbol x_i)$ and compute the $\phi(\boldsymbol x_i)^\top\phi(\boldsymbol x)$ with the underlying kernel function $\kappa(\boldsymbol x_i,\boldsymbol x)$.

# Compute the intercept in dual formulation

The dual SVM problem is:

$$
\min_{\boldsymbol\alpha}\frac{1}{2}\sum_{i=1}^m\sum_{j=1}^m\alpha_i\alpha_j y_i y_j \phi(\boldsymbol x_i)^\top\phi(\boldsymbol x_j)-\sum_{i=1}^m\alpha_i\,;\quad\text{s.t. }\sum_{i=1}^m\alpha_i y_i=0,\ 0 \le \alpha_i \le C\,.\tag{3}
$$

The Lagrangian is:

$$
\hat L(\boldsymbol\alpha,\beta,\boldsymbol\lambda,\boldsymbol\nu) = \frac{1}{2}\boldsymbol\alpha^\top\mathbf Q\boldsymbol\alpha - \boldsymbol\alpha^\top\mathbf 1+\beta\boldsymbol\alpha^\top\boldsymbol y-\boldsymbol\alpha^\top\boldsymbol\lambda+(\boldsymbol\alpha-C\mathbf 1)^\top\boldsymbol\nu\,,\tag{4}
$$

where $\lambda_i \ge 0$, $\nu_i \ge 0$, $\mathbf 1$ is an all-$1$ vector, $\mathbf Q$ is an $m \times m$ matrix such that $Q_{ij} = y_i\phi(\boldsymbol x_i)^\top\phi(\boldsymbol x_j)y_j$, and $\beta$ is actually the intercept.
We'll assume it for now, and reveal why it is in the end.
The KKT conditions are:

$$
\begin{cases}
\mathbf Q\boldsymbol\alpha=1-\beta\boldsymbol y+\boldsymbol\lambda-\boldsymbol\nu &\text{(stationarity)}\\
\lambda_i\alpha_i = 0 &\text{(complementary)}\\
\nu_i(C-\alpha_i) = 0 &\text{(complementary)}\\
\boldsymbol\alpha^\top\boldsymbol y = 0 &\text{(primal feasibility)}\\
0 \le \alpha_i \le C &\text{(primal feasibility)}\\
\lambda_i \ge 0 &\text{(dual feasibility)}\\
\nu_i \ge 0 &\text{(dual feasibility)}\\
\end{cases}\,.\tag{5}
$$

Thus, we have

$$
\begin{cases}
(\mathbf Q\boldsymbol\alpha)_i \ge 1 - \beta y_i &(\alpha_i=0)\\
(\mathbf Q\boldsymbol\alpha)_i \le 1 - \beta y_i &(\alpha_i=C)\\
(\mathbf Q\boldsymbol\alpha)_i = 1 - \beta y_i &(\text{otherwise})\\
\end{cases}\,.\tag{6}
$$

where $(\mathbf Q\boldsymbol\alpha)_i$ is the $i$th element of the vector $\mathbf Q\boldsymbol\alpha$.
When $S=\\{j \mid 0 < \alpha_j < C\\} \neq \varnothing$, for each such $j$,

$$
\beta = y_j(1 - (\mathbf Q\boldsymbol\alpha)_j)\,;
$$

which holds since $y_j = \pm 1$.
For numerical stability, we take the mean of all $\beta$'s as the final value of the intercept:

$$
\beta = \frac{1}{\\|S\\|}\sum_{j \in S} y_j (1 - (\mathbf Q\boldsymbol\alpha)_j)\,.
$$

When $S=\varnothing$, taking the first two cases of Equation $(6)$, it follows that

$$
\begin{cases}
\beta \ge 1-(\mathbf Q\boldsymbol\alpha)_i &(\alpha_i=0,y_i=1)\\
\beta \le -(1-(\mathbf Q\boldsymbol\alpha)_i) &(\alpha_i=0,y_i=-1)\\
\beta \le 1-(\mathbf Q\boldsymbol\alpha)_i &(\alpha_i=C,y_i=1)\\
\beta \ge -(1-(\mathbf Q\boldsymbol\alpha)_i) &(\alpha_i=C,y_i=-1)\\
\end{cases}\,.
$$

Equivalently, we have

$$
\max_{j \in T_1}\{y_j(1-(\mathbf Q\boldsymbol\alpha)_j)\} \le \beta \le \min_{j \in T_2}\{y_j(1-(\mathbf Q\boldsymbol\alpha)_j)\}\,,
$$

where

$$
\begin{cases}
T_1 = \{j \mid \alpha_j=0,y_j=1\text{ or }\alpha_j=C,y_j=-1\}\\
T_2 = \{j \mid \alpha_j=0,y_j=-1\text{ or }\alpha_j=C,y_j=1\}\\
\end{cases}\,,
$$

The intercept is taken as the mean of the lower and upper bounds.

## $\beta$ is the intercept

To show that $\beta$ is in fact the intercept in primal problem, we go further from Equation $(4)$, plugging in the stationarity conditions of Equation $(5)$, and it follows that

$$
\hat L(\boldsymbol\alpha,\beta,\boldsymbol\lambda,\boldsymbol\nu) = -\frac{1}{2}\boldsymbol\alpha^\top\mathbf Q\boldsymbol\alpha-C\mathbf 1^\top\boldsymbol\nu\,,
$$

where

$$
\boldsymbol\alpha=\mathbf Q^{-1}(1-\beta\boldsymbol y+\boldsymbol\lambda-\boldsymbol\nu)\,.
$$

assuming the inverse of $\mathbf Q$ exists.
Due to the structure of $\mathbf Q$, there exists a unique matrix $\mathbf Q^\frac{1}{2}$:

$$
Q^\frac{1}{2} =
\begin{pmatrix}
y_1\phi(\boldsymbol x_1) & \dots & y_m\phi(\boldsymbol x_m)\\
\end{pmatrix}
$$

such that $\mathbf Q=(\mathbf Q^\frac{1}{2})^\top\mathbf Q^\frac{1}{2}$.
Let $\boldsymbol w \triangleq \mathbf Q^{-\frac{1}{2}}(1-\beta\boldsymbol y+\boldsymbol\lambda-\boldsymbol\nu)=\mathbf Q^\frac{1}{2}\boldsymbol\alpha$.
The stationarity condition of Equation $(5)$ can be rewritten as:

$$
\begin{aligned}
\mathbf Q\boldsymbol\alpha &= 1-\beta\boldsymbol y+\boldsymbol\lambda-\boldsymbol\nu\\
(\mathbf Q^\frac{1}{2})^\top\boldsymbol w &= 1-\beta\boldsymbol y+\boldsymbol\lambda-\boldsymbol\nu\\
y_i\phi(\boldsymbol x_i)^\top\boldsymbol w+\beta y_i &\ge 1-\nu_i\quad\forall 1 \le i \le m\\
y_i(\phi(\boldsymbol x_i)^\top\boldsymbol w+\beta) &\ge 1-\nu_i\\
\end{aligned}
$$

Therefore, we have the dual of the dual problem as:

$$
\max_{\boldsymbol w,\beta,\boldsymbol\nu}-\frac{1}{2}\boldsymbol w^\top\boldsymbol w-C\mathbf 1^\top\boldsymbol\nu\,;\quad\text{s.t. }y_i(\phi(\boldsymbol x_i)^\top\boldsymbol w+\beta) \ge 1-\nu_i,\ \nu_i \ge 0\,.
$$

Clearly, $\beta$ is the intercept, and $\nu_i$ is the slack variable $\xi_i$ bounded to each sample in the dataset.

# Show that the two apporaches are equivalent

Recall that in primal formulation,

$$
\begin{aligned}
b &= y_j - \sum_{i=1}^m\alpha_i y_i \phi(\boldsymbol x_i)^\top\phi(\boldsymbol x_j) &\text{(primal formulation)}\\
b &= y_j (1-(\mathbf Q\boldsymbol\alpha)_j) &\text{(dual formulation)}\\
\end{aligned}
$$

If we plug in the definitions of $\boldsymbol w$ and $\mathbf Q$, it follows that

$$
\begin{aligned}
b &= y_j - \sum_{i=1}^m \alpha_i y_i \phi(\boldsymbol x_i)^\top\phi(\boldsymbol x_j)\\
b &= y_j (1 - y_j\sum_{i=1}^m \alpha_i y_i \phi(\boldsymbol x_i)^\top\phi(\boldsymbol x_j))\\
\end{aligned}
$$

However, since $y_j^2=1$, it can be easily shown that the two equations are the same.
