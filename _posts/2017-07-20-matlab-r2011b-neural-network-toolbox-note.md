---
layout: post
title:  "MATLAB R2011b 神经网络工具箱注意事项"
date:   2017-07-20 17:50:56 +0800
tags:   dev--matlab
---

这是记录了我使用神经网络工具箱时遇到的坑，供自己和他人参考。先写一点，以后遇到再更新。

## 1

```matlab
net = feedforwardnet;
net = train(net, attributes, targets);
```

第一行创建了一个两层前馈网络，隐藏层神经元个数为默认的10，这没什么问题。创建完网络后，如果使用 view(net) 来查看网络拓扑的话，会发现输入向量和输出向量是没有的，这是因为还没有调用 configure 函数。configure 函数默认在第一次调用 train 函数时被自动调用。**这里有一个坑**。假设：

```
X = [
  1 1 2;
  2 1 3;
  3 1 1;
  2 1 3]';
Y = [
  0 1 1 0];
```

即输入向量是3维向量，数据集X中包含4个样本，训练采用分批训练方式。经过 train 函数调用后，net.IW{1,1}的维度竟然会变成10x2！不应该是10x3吗（注：隐藏层神经元个数10，输入向量3维）？因为数据集X中所有样本的第二个属性都是一样的（值都是1），结果这个属性就被Matlab忽略掉了，不知是有意为之还是bug。解决方法

```matlab
X(:,find(var(X,0,1) < eps)) = X(:,find(var(X,0,1))) + min(min(X))*1e-5*randn(size(X,1),length(find(var(X,0,1))));
```

即，把被忽略的列加上一个小的白噪声让它们的值不一样。
