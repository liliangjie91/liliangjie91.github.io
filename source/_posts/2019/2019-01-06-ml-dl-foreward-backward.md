---
title: 古早博文-神经网络-前向传播与反向传播
date: 2019-12-31 21:47:36
categories: 
    - 技术
    - 机器学习
tags:
    - 机器学习
    - 神经网络
    - 深度学习
mathjax: true
---
### 概述
对于全连接的神经网络(MLP)，其结构看似复杂， 其实只是简单结构的不断重复。
这里的简单结构就是sigmoid函数即LR:
对于$x=(x_0,x_1,...,x_n)$ 有 $w=(w_0,w_1,...,w_n)$ 和 标量$b$ 使$$y=\sigma(wx+b)=\frac{1}{1+exp(\sum_{i=0}^{n}w_ix_i+b)}$$
$$z=wx+b$$
当然随着激活函数的发展，$\sigma$函数也在变化。这里先从一般情况介绍。
$\sigma(x)'=\sigma(x)(1-\sigma(x))$
这就是神经网络的一个神经元，它把上一层所有输出作为自己的输入，同时有自己的一套w

### 前向神经网络  
<!-- ![图1:简单神经网络](/image/image-network1.png) -->
<img src="/image/image-network1.png" width=512 height=256 />

如上图，
$x_0=a^0$表示第一层即输入层，接收一个样本的特征,他有$n_0$个神经元。
$a^1,a^2,...,a^{l-1}$表示隐层每一层的结果是上一层的输出也是下一层的输入，分别有$n_1,n_2,...,n_{l-1}$个神经元
$a^l=y'$表示输出层，当然，对于二分类，输出层是一个神经元，如果是多分类，就可以是多个神经元。有$n_l$个神经元
$W^l$表示第l-1层与l层之间的权重矩阵。$shape(W^l)=(n_l,n_{l-1})$
$b^l$表示第l层的偏至值。$shape(b^l)=shape(a^l)=(n_l,1)$
除第一层外其他层中每个圆即神经元，都是一个LR模型。
{% raw %}
$$
\begin{alignedat}{2}
a^0 &= x_0 \\
z^1&=W^1a^0+b^1 \\
a^1&=\sigma(z^1) \\
... \\
z^l&=W^la^{l-1}+b^l \\
a^l&=\sigma(z^l)=(\sigma(z^l_0),\sigma(z^l_1),...,\sigma(z^l_{n_l}))^T \\
y' &= a^L
\end{alignedat}$$
{% endraw %}
(这里的输出是通用输出，没按图1来。输出层神经元有$n_L$个,模型共L层)
上述的计算值，从输入层开始，逐层向前传播，经过隐层到达输出层。所以叫做前向传播


### 反向传播
反向传播用于计算每一层$W^l,b^l$的梯度，用于更新
现在模型建立好了，对于一个样本的输入，可以得到一个预测值$y'$。
现在需要设计一个损失函数来指导模型学习
$$C=f(y',y)$$
那有了损失函数后，就要求损失函数关于参数的梯度了，即需要求解
$\frac{\partial C}{\partial W^l},\frac{\partial C}{\partial b^l},l=1,2,...L$ 
{% raw %}
$$\begin{alignedat}{2}
\frac{\partial C}{\partial W^L}&=\frac{\partial C}{\partial z^L}\frac{\partial z^L}{\partial W^L} \\
&=(\frac{\partial C}{\partial a^L}\odot\frac{\partial a^L}{\partial z^L})\frac{\partial z^L}{\partial W^L} \\
&=(C'(a^L)\odot \sigma'(z^L))(a^{L-1})^T \\\\
\frac{\partial C}{\partial b^L}&=\frac{\partial C}{\partial z^L}\frac{\partial z^L}{\partial b^L} \\
&=C'(a^L)\odot \sigma'(z^L)
\end{alignedat}$$
{% endraw %}
其中
$$C'(a^l)=(C'(a^l_1),C'(a^l_2),...,C'(a^l_{nl}))^T$$
$$\frac{\partial a^l}{\partial z^l}=\sigma'(z^l) =(\sigma'(z^l_1),\sigma'(z^l_2),...,\sigma'(z^l_{n_l}) )^T $$
$$C'(a^l)\odot \sigma'(z^l)=(C'(a^l_1)\sigma'(z^l_1),C'(a^l_2)\sigma'(z^l_2),...,C'(a^l_{nl})\sigma'(z^l_{n_l}) )^T$$

我们发现，其实$\frac{\partial z^l}{\partial W^l}$是很容易求解的，而且求导过程与其他层没有关系。那么关键就是求$\frac{\partial C}{\partial z^l}$
我们令$$\delta^l=\frac{\partial C}{\partial z^l}$$
则$\delta^L=C'(a^L)\odot \sigma'(z^L)=(C'(a^l_L)\sigma'(z^l_L),C'(a^l_L)\sigma'(z^l_L),...,C'(a^l_{nL})\sigma'(z^l_{n_L}) )^T, shape=(n_L,1)$
如果已知$\delta^l$ 根据链式求导法则尤其是[链式向量求导](https://www.cnblogs.com/pinard/p/10825264.html)和[这个](https://blog.csdn.net/daaikuaichuan/article/details/80620518) (其实这里最重要的就是要确保链式求导的维度最终组合和初始的维度相同，那么对于向量求导就会涉及到转置和位置的交换。如果不觉得有问题，完全可以先对几个元素求导，然后总结出规律后来指导向量求导的组合)
我们首先约定一些东西。
* 每次的损失是一个标量，而W则是矩阵。当我们求$\frac{\partial C}{\partial W^l}$时，要保证结果和W的shape时一样的，即$shape(\frac{\partial C}{\partial W^l})=shape(W^l)=(n_l,n_{l-1})$
* $z^l,a^l,b^l$ 都是列向量
{% raw %}
$$\begin{alignedat}{2}
shape(\delta^l)&=(n_l,1)\\\\
\delta^{l-1}&=\frac{\partial C}{\partial z^{l-1}}=\frac{\partial C}{\partial z^l}\frac{\partial z^l}{\partial z^{l-1}}=\delta^l\frac{\partial z^l}{\partial z^{l-1}},shape=(n_{l-1},1)\\\\
\frac{\partial z^l}{\partial z^{l-1}}&=\frac{\partial z^l}{\partial a^{l-1}}\odot\frac{\partial a^{l-1}}{\partial z^{l-1}}=(W^{l})^T\odot\sigma'(z^{l-1}),shape=(n_{l-1},n_l) \\\\
so\\\\
\delta^{l-1}&=\delta^l((W^{l})^T\odot\sigma'(z^{l-1}))
\end{alignedat}$$
{% endraw %}
但是发现如上式那么组合的话，维度上是不相容的，所以应该调整成如下式：
$$\delta^{l-1}=((W^{l})^T\odot\sigma'(z^{l-1}))\delta^l=(W^{l})^T\delta^l\odot\sigma'(z^{l-1})$$
所以反向传播算法，关键记住如下几点：
  * $\delta^L=C'(a^L)\odot\sigma'(z^L)$
  * $\delta^l=(W^{l+1})^T\delta^{l+1}\odot\sigma'(z^l)$
  * $\frac{\partial C}{\partial W^l}=\delta^l (a^{l-1})^T$

  * $\frac{\partial C}{\partial b^l}=\delta^l $

### MLP 一般训练过程
####随机梯度下降：
对于一个样本(x,y)输入模型
首先，前向传播记录下每一层激活值$a^l$以及权重$W^l$
然后，计算$\delta^L$以及逐层$\delta^l$
然后，计算每层权重的梯度$\frac{\partial C}{\partial W^l},\frac{\partial C}{\partial b^l}$
最后，更新每层的权重
$$W^l=W^l-\eta\frac{\partial C}{\partial W^l}$$
$$b^l=W^l-\eta\frac{\partial C}{\partial b^l}$$

#### 批量梯度下降
对于m个样本(x,y)，依次输入模型
首先，前向传播记录下每一层激活值$a^l$以及权重$W^l$
然后，计算$\delta^L$以及逐层$\delta^l$
然后，计算每层权重的梯度$\frac{\partial C}{\partial W^l},\frac{\partial C}{\partial b^l}$
最后，更新每层的权重
$$W^l=W^l-\eta\sum\frac{\partial C}{\partial W^l}$$
$$b^l=W^l-\eta\sum\frac{\partial C}{\partial b^l}$$

### 不同损失函数与激活函数所带来的训练的不同
看不清楚请看：https://www.jianshu.com/p/1d6d7b4857d6

||$C=\frac{1}{2}(y'-y)^2$,$\sigma=sigmoid()$| $C=-ylogy'-(1-y)log(1-y')$,$\sigma=sigmoid()$|$C=-\sum y_ilogy_i'$，$\sigma^l=sigmoid(),\sigma^L=softmax()$|
|-|-|-|-|
|导数|$C'=y'-y,\sigma'=\sigma(1-\sigma)$|$C'=\frac{1-y}{1-y'}-\frac{y}{y'},\sigma'=\sigma(1-\sigma)$|$C'(a^L_i)=-\frac{1}{a^L_i} , softmax'(z^L_i)=a^L_i(1-a^L_i)$|
|$\delta^L=C'\odot\sigma'$|$(a^L-y)\odot\sigma(1-\sigma)$|$a^L-y$|$(0,0,...,a^L_i-1,..,0)^T$|
|$\frac{\partial C}{\partial W^L}$|$\delta^L(a^{L-1})^T$|$\delta^L(a^{L-1})^T$|$\delta^L(a^{L-1})^T$|
|$\delta^l$|$(W^{l+1})^T\delta^{l+1}\odot\sigma'(z^l)$|$(W^{l+1})^T\delta^{l+1}\odot\sigma'(z^l)$|$(W^{l+1})^T\delta^{l+1}\odot\sigma'(z^l)$|
|$\frac{\partial C}{\partial W^l}$|$\delta^l(a^{l-1})^T$|$\delta^l(a^{l-1})^T$|$\delta^l(a^{l-1})^T$|
|$\frac{\partial C}{\partial b^l}$|$\delta^l$|$\delta^l$|$\delta^l$|

对比前两列，最大的不同在$\delta^L$,使用交叉熵的模型少乘了一个$\sigma'$,而$\sigma'$往往是很小的(只在0附近比较大)，所以第二列会比第一列收敛快。

但关键是在$\delta^l$，大家都一样，但是随着l的不断减小，累乘的$\sigma'$越来越多，最后导致有的$\delta^l$越来越小趋近于0造成梯度消失(因为$0<sigmoid'\le0.25$)。这样导致底层网络权重得不到有效训练。同样，有的激活函数导数可能会很容易>1，这样就会造成梯度爆炸。总结起来就是，由于反向传播算法的固有缺陷，在网络层数过多时，会出现[梯度学习问题](https://blog.csdn.net/qq_25737169/article/details/78847691)，为了解决有如下常用方法，具体见上链接。
* 针对梯度爆炸，可以人为设定最大的梯度值，超过了就等于最大梯度值。这种做法叫**梯度剪切**。另外也可以对**权重做正则化**，来确保每次权重都不会太大。
* 针对梯度消失，如果激活函数的导数=1，那么就不会出现消失或爆炸，于是提出了ReLu激活函数
另外还有残差网络，batchnorm等技术

根本上就是针对BP的$\delta^l$的组成，要么从激活函数导数入手，要么从权重W入手，要么从连乘的传递结构入手等等。