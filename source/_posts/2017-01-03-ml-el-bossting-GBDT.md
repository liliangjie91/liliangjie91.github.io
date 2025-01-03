---
title: 古早博文-集成学习-Boosting-GBDT
date: 2017-12-31 20:04:05
categories: 
    - 技术
    - 机器学习
tags:
    - 机器学习
    - 集成学习
mathjax: true
---
#### GBDT （梯度提升树 Gradient Boosting Decision Tree ）
GBDT是以决策树（CART回归树）为弱学习器，应用加法模型的boosting算法
**模型**
$$f_M(x)=\sum_{t=1}^{M}h_t(x)$$
$$f_M(x)=f_{M-1}(x)+h_M(x)$$
M代表弱学习器个数，$h_t$ 是第t个弱学习器

**损失函数**
$$L(y, f_{t}(x)) =L(y, f_{t-1}(x)+ h_t(x))$$


**GBDT的基本思想**
$f_t(x)=f_{t-1}(x)+h_t(x)$
那么，新的$h_t(x)$应该怎么求解呢？
首先，新的$h_t(x)$需要减小损失 $L(y, f_{t}(x))$ ，那对于减小损失，我们肯定能想到梯度下降
注意，L是$f_t$ 的函数,其他参数y,x都是固定的，f(x)是L唯一可变的参数，不同之处在于f(x)是一个函数，而通常梯度下降的自变量是实数或实数向量，那么就可以求L关于f(x)的偏导/梯度
$$\frac{\partial L(y, f(x)))}{\partial f(x)} = \nabla L$$
即如果f(x)沿着上述负梯度方向走一步，就可以减小一部分损失
那么，f(x)走一步是什么意思？
因为$f_t(x)=f_{t-1}(x)+h_t(x)$，也就是说新走的这一步就是$h_t(x)$ ，
就像通常梯度下降更新参数 $\alpha_t = \alpha_{t-1} - \lambda \nabla L$ 一样
令 $f_t(x)=f_{t-1}(x)- \lambda\nabla L$
这里我们让$h_t(x)=- \nabla L$ （此处先忽略学习率$\lambda$，后续训练好$h_t$之后，可以再加上作为其系数）
所以，
**GB-Gradient Boosting** 是指下一个弱学习器拟合当前损失函数的负梯度。这是函数级的梯度下降法的一个应用。
而**DT-Decision Tree**，就是指弱学习器使用CART回归树：
$$h_t(x) = \sum\limits_{j=1}^{J}c_{tj}I(x \in R_{tj})$$

##### 回归算法
数据集
$$D=\{(x_1,y_1),(x_2,y_2), ...(x_m,y_m)\}$$
1，初始化弱学习器
$$f_0(x) = \underbrace{arg\; min}_{c}\sum\limits_{i=1}^{m}L(y_i, c)$$
2,对于后续1~M次的迭代：

* 2.1 对样本i=1,2，...m，计算负梯度
    $$r_{ti} = -\bigg[\frac{\partial L(y_i, f(x_i))}{\partial f(x_i)}\bigg]_{f(x) = f_{t-1}\;\; (x)}$$
    
    **得到新数据集**：
    $$D_t=\{(x_1,r_{t1}),(x_2,r_{t2}), ...(x_m,r_{tm})\}$$

* 使用数据集$D_t$拟合一棵CART树 $h_t(x)$
* 更新强学习器：
    $$
    \begin{alignedat}{}
    f_{t}(x) &= f_{t-1}(x) + h_t(x)\\
    &= f_{t-1}(x) + \sum\limits_{j=1}^{J}c_{tj}I(x \in R_{tj})\\
    \end{alignedat}$$
    其中，J为新树的叶子节点数，$c_{tj}$ 为新树第j个叶子节点的预测值

3,得到最终的回归器:
$$f(x) = f_M(x) =f_0(x) + \sum\limits_{t=1}^{T}\sum\limits_{j=1}^{J}c_{tj}I(x \in R_{tj})$$
此处的关键在于求解负梯度并用负梯度作为下一轮弱学习器需要拟合的值。
另外就是弱学习器的拟合过程，这就是决策树训练的问题了

##### 分类算法

GBDT应用于回归比较容易理解，因为本身CART回归树返回的就是连续值，在构造损失函数时，预测的连续值与真实的连续值方便计算损失，例如使用MSE，MAE等都很方便。
但是，当GBDT应用于分类任务时，真实值是0,1之类的离散标签，而回归树的返回值依旧是连续值，此时，应该如何构造损失函数呢？

其实，这样的问题，我们遇到过：即逻辑回归。
在逻辑回归中，$WX$ 是一个线性模型，其返回的也是连续值，但可以通过sigmod函数将其转换成一个0-1的概率值
$$P(y=1|x)=\frac{1}{1+exp(-wx)}$$
同样，我们也可以把GBDT最终的强分类器转换成逻辑回归的形式即
$$
\begin{alignedat}{2}
P(y=1|x)&=\frac{1}{1+exp(-f_M(x))}\\
&=\frac{1}{1+exp(-\sum_{j=0}^{M}h_j(x))}\\
&=s(f_M(x))\\\\
P(y=0|x)&=1-\frac{1}{1+exp(-f_M(x))}\\
&=\frac{1}{1+exp(f_M(x))}
\end{alignedat}
$$

则，对于样本$(x_i,y_i)$
分对的概率
$$P_i=P(y=1)^yP(y=0)^{(1-y)}$$
利用对数最大似然做损失函数(目标函数)
$$
\begin{alignedat}{2}
L&=log(\sum_{i=1}^{m}Pi)\\
&=\sum_{i=1}^{m}\bigg[y_ilogP(y=1)+(1-y_i)logP(y=0)\bigg]\\
&=\sum_{i=1}^{m}\bigg[y_if_M(x_i)-log(1+exp(f_M(x)))\bigg]\\\\
\nabla L &=\frac{\partial L}{\partial f_M(x_i)}\\
&=y_i - s(f_M(x_i))
\end{alignedat}$$

因为是最大化上述L，所以此处选择正梯度。
即，下一个弱学习器$h_t(x_i)$要拟合 $y_i - s(f_{t-1}(x_i))$

##### 正则
1，缩减系数 
$f_{k}(x) = f_{k-1}(x) + \nu h_k(x) ,0 < \nu \leq 1$
类似于梯度下降中的学习率$\lambda$，
也类似于adaboost中的弱学习器权重 $\alpha$
2，子采样
3，剪枝


#### 提升树 Boosting Tress
GBDT就是一种提升树。但此处为何后讲提升树先讲GBDT呢？
因为如果一开始讲BT，就难以绕开一个比较经典的例子：预测年龄
对于预测年龄这个回归问题，在加法模型这里可以这么做：
一个30岁的人，先预测20岁作为第一个弱模型M1
M1略有不足，有多少不足？有30-20=10的不足。
于是下一个弱模型M2就把10岁当做新数据去拟合预测，得到7
这样M1+M2=27，比较准确了，但依旧有3的不足
于是下一个弱模型M3去预测3
以此类推，得到最终的强模型$f=M1+M2+...+Mn$

这个例子，其实在一定程度上解释了大部分boosting模型，尤其是加法模型。
即：每个弱学习器只拟合模型目前的不足。

其实，弱学习器拟合很容易，关键是，如何度量这个“不足”
AdaBoost用改变样本权重的方式度量当前模型的不足，权重的改变会引起错误率$e$的改变
GBDT用损失函数的负梯度来度量这个不足。
而有些模型，会使用一个叫“残差”的东西度量不足。
这个模型出现在以**平方损失**为损失函数，以提升树为基本模型的**回归任务**中。
具体的
$$
\begin{alignedat}{2}
L&=(y-\widehat{y})^2\\
&=(y-f_{t-1}(x)-h_t(x))^2\\
&=(r - h_t(x))^2\\
r&=y-f_{t-1}(x)
\end{alignedat}$$
在这里，残差r就是预测值与样本值之差
从上述L来看，如果要让L减小，就应该让 $h_t(x)$ 尽可能去拟合 r 没毛病。
而注意到，L在$f_{t-1}$处的负梯度，**恰好**也等于r
所以，这个模型，也可以是GBDT。
那**是不是所有负梯度都等于残差r呢？** 肯定不是，只有平方损失才是
那两者不相等，**到底应该去拟合负梯度还是残差呢？**
一直都是拟合负梯度，只不过平方损失时负梯度有另一个名字叫“残差”
**那我可不可以只去拟合残差即便损失函数不用平方损失的时候？因为计算残差很简单啊！**
~~1，在分类问题中，无法直接使用残差。就像上述GBDT的分类问题中，需要把强学习器转换成概率而这时如果用残差是无意义的。~~
以下内容，部分引用自知乎：
1，在机器学习算法中，总要有一个损失函数，我们的目的是减小损失函数。
如果是用了残差，即每一轮都去直接拟合上一轮残差，也就是说放弃了损失函数的指导，即此时无论选择什么损失函数，模型不会有什么区别。
损失函数减小和残差减小可以乍一看方向一样，但是细想还是有很大区别的。即我们训练一个模型，目的是为了去预测~~位置~~未知数据，这里就涉及到模型的泛化能力，所以训练的模型不能过拟合。为了防止过拟合，损失函数可以添加正则项，而残差无法添加正则项，如果一味拟合残差，容易过拟合
2，其实，当你选择了拟合残差，就是选择了平方损失做损失函数。而损失函数不止一种。
3，另外我不是很喜欢“损失函数负梯度去拟合残差”这句话，很容易引起误解，觉得残差才是最好的那个。不同的损失函数应对不同的模型和任务，残差或许适合某些任务，但没有GBDT~~广发~~广泛。
4，或许我们对残差的理解有误。我更倾向于把残差改为“当前模型的不足”这个不足不(只)是针对训练数据的不足，而是针对完备的真实数据的不足。而损失函数，尤其是带正则的损失函数，更能度量模型与完备真实数据的偏差。

#### 讨论
>优点：我们可以把树的生成过程理解成自动进行多维度的特征组合的过程，从根结点到叶子节点上的整个路径(多个特征值判断)，才能最终决定一棵树的预测值。另外，对于连续型特征的处理，GBDT 可以拆分出一个临界阈值，比如大于 0.027 走左子树，小于等于 0.027（或者 default 值）走右子树，这样很好的规避了人工离散化的问题。

>缺点：对于海量的 id 类特征，GBDT 由于树的深度和棵树限制（防止过拟合），不能有效的存储；另外海量特征在也会存在性能瓶颈，经笔者测试，当 GBDT 的 one hot 特征大于 10 万维时，就必须做分布式的训练才能保证不爆内存。所以 GBDT 通常配合少量的反馈 CTR 特征来表达，这样虽然具有一定的范化能力，但是同时会有信息损失，对于头部资源不能有效的表达。
#### 参考
[刘建平博客-GBDT](https://www.cnblogs.com/pinard/p/6140514.html#!comments)
[GBDT用于分类](https://zhuanlan.zhihu.com/p/46445201)
统计学习方法-李航
[GBDT缩减系数](https://blog.csdn.net/ningyanggege/article/details/87974691)
[负梯度与残差](https://blog.csdn.net/zbzckaiA/article/details/86235978)
[https://cloud.tencent.com/developer/article/1005416](https://cloud.tencent.com/developer/article/1005416)



