---
layout: post
title:  "被截短的随机分布与原分布的关系"
date:   2017-04-24 00:29:35 +0800
tags:   math--prob
---

已知随机分布的概率密度函数为$f_X(x)$，定义域为$D$。现将其定义域截取为$E$，其中$E \subseteq D$，即不断按照该分布取随机变量直到变量值落在$E$中。截取后的随机变量的分布的概率密度函数与$f_X(x)$是什么关系呢？

要回答这个问题，首先设截取后的概率密度函数为$f_U(x)$，设$a=\min{E}$（如果$E$无下界，令$a$表示$-\infty$）。$\forall x \in E$：

$$
\begin{aligned}
\int_a^x{f_U(t)\mathrm{d}t} &= \int_a^x{f_X(t)\mathrm{d}t} + \left(1 - \int_E{f_X(t)\mathrm{d}t}\right)\int_a^x{f_X(t)\mathrm{d}t} + \cdots\\
\int_a^x{f_U(t)\mathrm{d}t} &= \sum_{n=1}^\infty{\left(1-\int_E{f_X(t)\mathrm{d}t}\right)}^n \int_a^x{f_X(t)\mathrm{d}t}\\
\int_a^x{f_U(t)\mathrm{d}t} &= \left(\int_E{f_X(t)\mathrm{d}t}\right)^{-1} \int_a^x{f_X(t)\mathrm{d}t}\\
{\mathrm{d} \over \mathrm{d}x}\int_a^x{f_U(t)\mathrm{d}t} &= \left(\int_E{f_X(t)\mathrm{d}t}\right)^{-1} {d \over \mathrm{d}x}\int_a^x{f_X(t)\mathrm{d}t}\\
f_U(x) &= \left(\int_E{f_X(t)\mathrm{d}t}\right)^{-1} f_X(x)
\end{aligned}
$$

所以随机分布在形状上不会有什么改变，但会变高。
