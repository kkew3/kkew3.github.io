---
layout: post
title:  "如何尽可能快地确定宝可梦属性"
date:   2023-04-26 18:12:07 +0800
categories: algorithm
---

# 确定宝可梦属性的方法

可用宝可梦对攻击的反应确定宝可梦的属性.
单一属性宝可梦对单一属性攻击的反应有以下四种: 无效, 抵抗, 一般, 有效; 可用乘数 0, 1/2, 1, 2 表示.
双属性宝可梦对单一属性攻击的反应为以上四个乘数的两两相乘的结果, 分别为 0, 1/4, 1/2, 1, 2, 4, 即无效, 非常抵抗, 抵抗, 一般, 有效, 非常有效.
用乘法表示属性乘数的叠加不是很方便, 故对乘数取底数为 2 的对数, 变为 $-\infty$, -2, -1, 0, 1, 2 六种反应, 下文会使这样操作方便的原因变得显而易见.

# 数学表示

给定属性克制矩阵 $\mathbf A$, 其中第 $i$ 行第 $j$ 列的元素 $a_{ij} \in \{-\infty, -2, -1, 0, 1, 2\}$ 表示单一属性为 $j$ 的宝可梦对属性为 $i$ 的攻击的对数抵抗乘数.
因为一共有 18 种属性, 所以 $\mathbf A$ 的维度为 $18 \times 18$.
使用 one-hot encoding 以及其加性叠加表示宝可梦的单一及双属性, 向量为 18 维;
例如 $(0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0)^\top$ 表示宝可梦具有第 2 种和第 4 种属性, 为双属性宝可梦.
令宝可梦属性矩阵 $\mathbf Q$ 的每一列表示一种宝可梦属性的枚举; 由于所有单一属性和双属性的个数为 $18 + \binom{18}{2} = 171$, 故其维度为 $18 \times 171$.
令 $18 \times 18$ 单位矩阵 $\mathbf I$ 的第 $i$ 列为 $\boldsymbol e_i$.

由于双属性宝可梦对攻击的对数抵抗乘数的叠加是加性的, 因此可用矩阵乘法自然地表示.
例如,

$$
\boldsymbol r = \boldsymbol e_i^\top \mathbf A \mathbf Q
$$

表示在第 $i$ 种属性的攻击下各属性宝可梦的对数抵抗乘数.
如果各乘数在 $\boldsymbol r$ 中是唯一的, 那么便可唯一地确定宝可梦的属性.
即使只有一个元素子集中的乘数唯一, 也能排除掉这些宝可梦属性, 以便进一步确定.

# 确定宝可梦属性的算法

令 $s$ 为 $1,\dots,18$ 的一个排列, 使得第 $j$ 次尝试使用属性为 $s(j)$ 的攻击.
确定宝可梦属性的解即为形似 $s$ 的一个排列.
显然, 暴力枚举具有 $O(n!)$ 复杂度, 不可行.
我们可使用贪心策略确定宝可梦属性.

初始化剩余宝可梦属性矩阵 $\mathbf Q^{(0)} = \mathbf Q$, 已尝试过的攻击属性集合 $T^{(0)} = \varnothing$, 已确定的攻击序列为 $s^{(0)} = ()$.
假设在第 $k$ 次尝试前, 剩余宝可梦属性矩阵为 $\mathbf Q^{(k-1)}$, 其为原宝可梦属性矩阵 $\mathbf Q$ 的列的子集; 已尝试过的攻击属性集合为 $T^{(k-1)}$, 其元素属于 $T = \{1,\dots,18\}$; 已确定的攻击序列为 $s^{(k-1)}$.
如果 $\mathbf Q^{(k-1)}$ 的列数为零, 算法结束.
否则, $\forall i \in T \,\backslash\, T^{(k)}$, 计算 $\boldsymbol r_i = \boldsymbol e_i^\top \mathbf A \mathbf Q^{(k)}$, 使得 $\boldsymbol r_i$ 中的重复元素数目最小, 令所对应的 $i$ 为 $i^\ast$.
令 $s^{(k)} = s^{(k-1)} \cup i^\ast$, $T^{(k)} = T^{(k-1)} \cup \{i^\ast\}$, $Q^{(k)}$等于去掉在 $\boldsymbol r_{i^\ast}$ 中元素唯一的列的 $Q^{(k-1)}$.

算法实现时可用一足够小的负数, 例如 -20, 表示 $-\infty$, 然后在 $\boldsymbol r$ 中把所有足够小的数重置为 -20, 以模拟 $-\infty$ 加减任何数 (注意我们的 operand 集合) 都为其本身.

Python 实现:

```python
import numpy as np

n = 18

def calc_r(i, A, Q):
    r = np.eye(n, dtype=int)[i:i+1].dot(A).dot(Q)
    r[r < -5] = -20
    return r

def shrink_Q(r, Q):
    _, v, c = np.unique(r, return_inverse=True, return_counts=True, axis=1)
    return Q[:, c[v] > 1]

A = ...
Q = ...

def greedy():
    s = []
    T = set(range(n))
    while Q.shape[1] > 0:
        best_i = None
        min_next_Q = Q
        for i in T:
            next_Q = shrink_Q(calc_r(i, A, Q), Q)
            if next_Q.shape[1] < min_next_Q.shape[1]:
                min_next_Q = next_Q
                best_i = i
        Q = min_next_Q
        s.append(best_i)
        T.remove(best_i)
    return s
```

# 原问题的扩展

通过对算法简单的扩展, 还能回答以下问题:

- 已知宝可梦具有某种属性, 想确定其是否具有第二属性, 如果有, 是什么属性: 通过移除矩阵 $\mathbf Q$ 的相应列解决
- 希望只使用具有某些属性的攻击确定宝可梦的属性: 通过移除集合 $T$ 的相应元素解决
