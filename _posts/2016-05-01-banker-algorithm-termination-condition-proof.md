---
layout: post
title:  "银行家算法结束条件的合理性证明"
date:   2016-05-01 10:11:59 +0800
categories: algorithm
---

首先简要提一下银行家算法的流程（类Java的伪代码）。算法的具体说明请参见操作系统课本。

```java
/*
 * 令work为长为m的数组，表示m种资源的剩余量；
 * 令finish为长为n的布尔数组，表示n个进程是否已经结束；
 * 令need为n行m列的二维数组，need[i]表示第i个进程在当前时刻所需的最大资源量；
 * 令allocation为n行m列的二维数组，allocation[i]，表示第i个进程在当前时刻已被分配的资源量。
 * 令available为长为m的数组，表示初始可用的资源量
 *
 * array1 ope array2 表示两数组长度（记为len）相等，且对于任意0 <= i < len，array1[i] ope array2[i]。
 * 例如 array1 < array2表示对于任意0 <= i < len，array1[i] < array2[i]。
 */
work = available;
for (int i = 0; i < finish.length; i++)
        finish[i] = false;
while there exists such an i that
finish[i] == false && need[i] <= work
        work -= allocation[i];
        finish[i] = true;
for (int i = 0; i < finish.length; i++)
        if (finish[i] = false)
                return false; //可能发生死锁
return true; //不可能发生死锁
```

不知有没有人会质疑该算法的结束条件：该算法没有回溯过程，如何保证这次没有找到一个进程运行的安全序列，这n个进程的任意顺序排列就都不可能构成安全序列呢？

证明如下：

假设有$n$个进程，以序号表示为

$$
[1, 2, \dots, n]
$$

进程运行序列进行到

$$
S = [i_1, i_2, \dots, i_k]\ (k < n)
$$

时无法继续算法（即不能找出一个i满足finish[i]==false && need[i] <= work），被判定为可能发生死锁。

令集合$C = \{i_1, i_2, \dots, i_k\}$；并令集合$D$为集合$\{1, 2, \dots, n\}$与$C$的差集，即所有`finish`为`false`的进程所组成的集合。

如果此时无法继续算法，那么根据算法流程，

$$
\min_{j\in D}\big\{\text{need}_j\big\} > \text{available} + \sum_{j \in C}\text{allocation}_j
$$

若存在另一个序列$S'$，使得$S'$为安全序列，则$S'$中的元素排列只能为以下情况之一：

1. 前$k$个元素构成的集合与$C$相同（但排列顺序可能不同），且后$n-k$个元素构成的集合与$D$相同（但排列顺序可能不同）；

2. 前$k$个元素中至少有一个元素属于集合$D$，且后$n-k$个元素中至少有一个元素属于集合$C$。

对于第一种情况，根据（命题1）不应存在；

对于第二种情况，假设某一个属于集合$D$的元素出现在$S'$的第$t$（$1\le t\le k$）个位置上。令集合$C'$为集合$C$中的前$t$个元素构成的集合，那么此时应有

$$
\exists j \in D,\ \text{need}_j \le \text{available} + \sum_{j \in C'}\text{allocation}_j\quad\text{(命题2)}
$$

由于$C'$是$C$的子集，所以命题2中的和式一定不大于命题1中的和式。因此如果命题1是正确的，那么命题2一定是错误的。

所以，只要存在一个序列不是安全序列，那么这$n$个进程的任意排列都不是安全序列。

换言之，只要有一个序列是安全序列，那么在算法进行过程中出现的任何分叉点所构成的其它序列就都是安全序列。
