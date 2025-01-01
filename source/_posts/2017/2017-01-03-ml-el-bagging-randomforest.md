---
title: 古早博文-集成学习-Bagging-随机森林RF
date: 2017-12-31 20:01:25
categories: 
    - 技术
    - 机器学习
tags:
    - 机器学习
    - 集成学习
mathjax: true
---
#### bagging

bagging又名Bootstrap aggregating(自助聚合法，很扯的翻译)
Bootstrap 又名自助法，是统计学上的概念，核心思想是**样本重采样(有放回)**，*重采样来干啥，具体是每次重采样获取子数据集--一个估计；多次重采样得到多个估计，这就可以计算估计的方差等统计量。在这里我们无需在乎。*
aggregating 聚合。即多模型聚合成一个模型。

**大致流程**
1，对样本集D有放回地随机重采样成m个子样本集D1，D2，...Dm
2，对于每个子样本集Di,训练一个弱学习器Mi
3，综合m个学习器，对于分类则投票，对于回归，则均值

#### 随机森林 RF

bagging的一种算法。
1，弱学习器指定CART树
2，除了样本随机之外，特征也随机选取。


优点：
**随机采样+随机特征+多模型平均 可以充分减小模型方差**
可以并行运行
对于高纬度特征也可以快速计算 


#### bagging 与 方差

直观讲，投票(分类)与平均(回归)本身就是一种相对稳定可以对抗高方差的方式。

具体来讲，bagging的做法，是随机重采样获取n个子样本集$D_i$ 在对每个子样本集训练模型(使用同一个算法训练)$M_i$ 则$M_i$就会有相似的均值与方差
最终的到的模型
$$M=\frac{1}{n}\sum_{i=1}^{n}M_i$$
**模型的期望**
$$
\begin{alignedat}{}
E(M)&=E(\frac{1}{n}\sum_{i=1}^{n}M_i)\\
&=\frac{1}{n}E(\sum_{i=1}^{n}M_i)\\
&=\frac{1}{n}nE(M_i)\\
&=E(M_i)
\end{alignedat}$$
即bagging对bias的影响较小

$$
\begin{alignedat}{}
Var(M)&=Var(\frac{1}{n}\sum_{i=1}^{n}M_i)\\
&=\frac{1}{n^2}Var(\sum_{i=1}^{n}M_i)\\
\end{alignedat}$$

**模型的方差**
对于方差有
$$Var(aX+bY)=a^2Var(X)+b^2Var(Y)+2cov(X,Y)$$
X,Y相互独立时 $cov(X,Y)=0$

1，当所有$M_i$都相互独立时
$$
\begin{alignedat}{2}
Var(M)&=\frac{1}{n^2}Var(\sum_{i=1}^{n}M_i)\\
&=\frac{1}{n^2}nVar(M_i)\\
&=\frac{Var(M_i)}{n}
\end{alignedat}$$

2，当所有$M_i$完全不独立，即所有模型相等时
$$
\begin{alignedat}{2}
Var(M)&=\frac{1}{n^2}Var(\sum_{i=1}^{n}M_i)\\
&=\frac{1}{n^2}Var(nM_i)\\
&=Var(M_i)
\end{alignedat}$$

而在bagging中，M介于上述两种情况之间即
$$Var(M_i)>Var(M)>\frac{Var(M_i)}{n}$$
所以bagging的最终模型方差会减小。



#### 参考
[bagging与方差](https://www.zhihu.com/question/26760839)
[刘建平博客-bagging](https://www.cnblogs.com/pinard/p/6156009.html)