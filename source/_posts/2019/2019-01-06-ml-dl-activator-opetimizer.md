---
title: 古早博文-神经网络-激活函数与优化器
date: 2019-12-31 21:51:17
categories: 
    - 技术
    - 机器学习
tags:
    - 机器学习
    - 神经网络
    - 深度学习
mathjax: true
---
### 1，激活函数
具体激活函数参见此篇
[https://www.jiqizhixin.com/articles/2017-10-10-3](https://www.jiqizhixin.com/articles/2017-10-10-3)

激活函数从图像上来看通常比较简单。他的工作内容也很简单，就是对上一层输出$a^{l-1}$与本层权重$W^l$的加权求和值$z^l$做一个转换变成$a^l$。通常这个转换是非线性的。
那不要这个激活函数好不好？很容易证明，如果不用激活函数，神经网络会退化成简单线性函数：
$$
\begin{alignedat}{2}
a^0&= x_0 \\\\
z^1&=W^1a^0+b^1 \\\\
a^1&=z^1 \\\\
z^2&=W^2z^1+b^2=W^2W^1a^0+W^2b^1+b^2 \\\\
z^3&=W^3z^2+b^3=W^3W^2W^1a^0+W^3W^2b^1+W^3b^2+b^3 \\\\
... \\\\
z^l&=W^lz^{l-1}+b^l=\prod^{l}_{k=1}W^kx_0+fun(b)=Ax_0+b  \\\\
\end{alignedat}
$$
这相当于千辛万苦设计了一个网络，最后一看原来只是一组线性函数。隐层亦失去了意义。
而有了激活函数就不一样了，激活函数通常是非线性的，相当于为神经网络引入了非线性，上述式子就无法简化成简单线性函数：
{% raw %}
$$
\begin{alignedat}{2}
a^0 &= x_0 \\
z^1&=W^1a^0+b^1 \\
a^1&=\sigma(z^1) \\
... \\
z^l&=W^la^{l-1}+b^l \\
&=(z^l_0,z^l_1,...,z^l_{n_l})^T\\\\
a^l&=\sigma(z^l)=(\sigma(z^l_0),\sigma(z^l_1),...,\sigma(z^l_{n_l}))^T \\
\end{alignedat}$$
{% endraw %}
这是神经网络具有强大拟合能力的根源。
对比上述两个$z^l$ 
无激活函数的z实际上是只对输入做了一次线性转换
有激活函数的z实际上是对输入做了多次非线性转换，不同的激活函数，则又在非线性转换上产生不同的作用。

### 2，优化器 [Optimizer](https://www.cnblogs.com/guoyaohua/p/8542554.html)
对于神经网络的损失优化，通常使用的是随机梯度下降SGD或批量梯度下降BGD或mini-batch梯度下降MBGD。梯度下降确实是一个根本，但是在每步梯度下降的步长(学习率learning rate)及其变化以及梯度下降方向的选择上则产生了很多变种。
具体参见[此篇](https://www.cnblogs.com/guoyaohua/p/8542554.html)
优化器是什么？
是用来一步步优化参数W和b的。给一批样本，经过模型得到一个损失，通过损失反向求出各个参数对应的梯度，然后用本轮Ｗ减去这个（梯度乘以步长）得到新的W和b。这样一步步迭代，直到得到一个比较好的结果。这就是梯度下降的朴素思想。
然而实际情况却没有这么好。有许多乌云笼罩在这一步：
* 指导思想有了，但我要求计算速度要快--SGD
* 计算速度快了，但我要求准确度不能太差--MBGD

这些还好说，更严重的：
* 鞍点处各个梯度也为0，走不动了，但这里不是最优点啊。
* 学习率太小了，一步步挪，什么时候才能到最优点啊
* 学习率太大了，一步跨过了最优点，然后反向跨又跨过了又得反向，来回震荡，停不下来啦
* 通向最优点的方向梯度较小较平缓，而垂直于此方向的梯度却很大，且波澜起伏，导致我在这个方向来回震荡而往最优点的方向走的很慢，浪费时间
* 虽然看似每次都更新W，但实际上，每次只是更新W的一部分而非全部。那么有的w更新频次就多，有的就低。那使用同样的学习率，必然导致更新频次低的w们没有到达最优值。

具体就参见上面的链接吧，总结的非常好了
[yhq.gif](https://upload-images.jianshu.io/upload_images/3376541-e4682f4503b16cdc.gif?imageMogr2/auto-orient/strip)
[yhq2.gif](https://upload-images.jianshu.io/upload_images/3376541-f391a40094a0f0a3.gif?imageMogr2/auto-orient/strip)

### 3，参考
[https://www.cnblogs.com/guoyaohua/p/8542554.html](https://www.cnblogs.com/guoyaohua/p/8542554.html)
[https://blog.csdn.net/tyhj_sf/article/details/79932893](https://blog.csdn.net/tyhj_sf/article/details/79932893)
[https://www.jiqizhixin.com/articles/2017-10-10-3](https://www.jiqizhixin.com/articles/2017-10-10-3)

