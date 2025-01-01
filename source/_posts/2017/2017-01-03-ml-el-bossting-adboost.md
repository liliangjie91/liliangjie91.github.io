---
title: 古早博文-集成学习-Boosting-Adaboost
date: 2017-12-31 20:02:28
categories: 
    - 技术
    - 机器学习
tags:
    - 机器学习
    - 集成学习
mathjax: true
---
##### 0.Adaboost介绍
**Adaboost**是以
加法模型为模型，
前项分布算法为学习算法，
指数损失函数为损失函数的boosting集成学习算法。

**所谓加法模型**，即最终的强分类器是由多个弱分类器加权求和所得
**所谓前项分布算法**，即第k+1个强分类器是由第k个强分类器学成后，再学习一个弱分类器组合而得。
**所谓指数损失函数**，即 
$\sum_{i=1}^{m}\exp({-y_{i}f_{k}(x))}$ 

好处是，指数中幂相加可以转化为指数相乘，这样，可以结合前项分布算法提取出固定的部分，从而**简化计算**。
当然，可以选择其他损失函数，那就是其他类型的boosting算法了

##### 1.流程：
训练集 
{% raw %}
$$
T= \left \{ (x^{(1)},y^{(1)}),(x^{(2)},y^{(2)}),...(x^{(m)},y^{(m)}) \right \}
$$
{% endraw %}
此处　　$$y={-1 ,1}$$

训练集样本的权重
$$
W(k)=(w_{k1},w_{k2} ... w_{km}),w_{1i}=\frac{1}{m} ,i=1,2...m
$$
注意，初始权重即1/m ，如有1000个样本，初始权重即 w=1/1000

第k个弱分类器加权误差：
$$
e_{k}= \sum_{i=1}^{m}w_{ki}I(G_{k}(x_{i})\neq y_{i})
$$
注意，这里分类加权误差，即本次样本集错分样本权重之和，是归一化后的。

第k个弱分类器权重：
$$
\alpha _{k} = \frac{1}{2}log(\frac{1-e_{k}}{e_{k}})
$$
理论上，弱分类器误差ek应该是<0.5的，则alpha > 0
ek越小，alpha越大
即，弱分类器效果越好，则其权重越大。

第k+1个弱分类器分类时，样本的权重：
{% raw %}
$$
\begin{alignedat}{}
w_{k+1i}&=\frac{w_{ki}}{Z_{k}}exp(-\alpha _{k}y_{i}G_{k}(x_{i})) \\
Z_{k}&=\sum_{i=1}^{m}w_{ki}exp(-\alpha _{k}y_{i}G_{k}(x_{i}))
\end{alignedat}
$$
{% endraw %}
$Z_{k}$是个固定值，alpha默认>0
所以，
如果样本i分错了，则exp(...)的值>1
* 即相比于上一次的弱分类，样本i的权重在本次弱分类中得到了**提高**。

如果样本i分对了，则exp(...)的值<1
* 即相比于上一次的弱分类，样本i的权重在本次弱分类中得到了**降低**。

所以，adaboost的思想就通过数学公式表达了出来：

1. 弱分类器错误率越低，则权重alpha越高

2. 样本越被分错，则权重w越高（具体的上一次弱分类分错的样本，在下一次弱分类时权重会提高，而如何提高则会根据上一个弱分类器的权重alpha来确定。）


弱分类器集合策略：加法模型
{% raw %}
$$
\begin{alignedat}{2}
f_{K}(x)&=\sum_{k=1}^{K}\alpha _{k}G_{k}(x))\\
\end{alignedat}
$$
{% endraw %}
但会疑问，权重更新公式具体是怎么来的？为什么看着很复杂？

##### 2.推导

adaboost的损失函数用的是指数损失函数
$$\sum_{i=1}^{m}\exp({-y_{i}f_{k}(x))}$$
这个损失是有道理的，
如果分错，则损失>1，且相差越多，损失越大
如果分对，则损失<1，且相差越多，损失越大

则adaboost的损失函数
{% raw %}
$$
\begin{alignedat}{}
L
&= \sum_{i=1}^{m} exp(-f_{k}(x_{i})y_{i}) \\
&=\sum_{i=1}^{m} exp(-(f_{k-1}(x_{i})y_{i}+\alpha_{k}y_{i}G_{k}(x_{i}))) \\
&= \sum_{i=1}^{m} (exp(-f_{k-1}(x_{i})y_{i})exp(-\alpha_{k}y_{i}G_{k}(x_{i}))) \\
\end{alignedat}
$$
{% endraw %}
令

$$
w_{ki}^{'}=exp(-f_{k-1}(x_{i})y_{i})
$$
则
{% raw %}
$$
\begin{alignedat}{2}
L
& = \sum_{i=1}^{m} (exp(-f_{k-1}(x_{i})y_{i})exp(-\alpha_{k}y_{i}G_{k}(x_{i}))) \\
& = \sum_{i=1}^{m} (w_{ki}^{'}exp(-\alpha_{k}y_{i}G_{k}(x_{i}))) \\
& = e^{\alpha} \sum_{y_{i}\neq G_{k}(x_{i})}w_{ki}^{'} + 
e^{-\alpha} \sum_{y_{i}=G_{k}(x_{i})}w_{ki}^{'} \\
& = \sum_{i=1}^{m}w_{ki}^{'} (e^{\alpha} \frac{\sum_{y_{i}\neq G_{k}(x_{i})}w_{ki}^{'}}{\sum_{i=1}^{m}w_{ki}^{'}}+e^{-\alpha} \frac{\sum_{y_{i}=G_{k}(x_{i})}w_{ki}^{'}}{\sum_{i=1}^{m}w_{ki}^{'}}) \\
\end{alignedat}
$$
{% endraw %}
令 
$$
err^{'}=\frac{\sum_{y_{i}\neq G_{k}(x_{i})}w_{ki}^{'}}{\sum_{i=1}^{m}w_{ki}^{'}}
$$
则
{% raw %}
$$
\begin{alignedat}{2}
L
&=\sum_{i=1}^{m}w_{ki}^{'} (e^{\alpha}err^{'}+e^{-\alpha}(1-err^{'})) \\
&=\sum_{i=1}^{m}w_{ki}^{'} ((e^{\alpha}-e^{-\alpha})err^{'}+e^{-\alpha}) 
\end{alignedat}
$$
{% endraw %}
为求alpha，令
{% raw %}
$$
\begin{alignedat}{2}
\frac{\partial L}{\partial \alpha} &=0 \\
\frac{\partial L}{\partial \alpha} &=
\sum w_{ki}^{'} ((e^{\alpha}+e^{-\alpha})err^{'}-e^{-\alpha}) \\\\
\Rightarrow  \\
((e^{\alpha}&+e^{-\alpha})err^{'}-e^{-\alpha}) =0 \\\\
\Rightarrow \\
\alpha &= \frac{1}{2}log(\frac{1-err_{'}}{err_{'}})
\end{alignedat}
$$
{% endraw %}
可以看到，此处的alpha和上面流程中的弱分类器权重alpha形式一样。只需证明
err｀=e即可

##### 3.证明 err'=e
注意
{% raw %}
$$
\begin{alignedat}{}
err^{'}&=\frac{\sum_{y_{i}\neq G_{k}(x_{i})}w_{ki}^{'}}{\sum_{i=1}^{m}w_{ki}^{'}} \\
w_{ki}^{'}&=exp(-f_{k-1}(x_{i})y_{i})
\end{alignedat}
$$
{% endraw %}
可以看出err是一个类似于错误率的东西，表示w·总体中错分的那部分w·
则关键在于，w·是否等价于样本权重w

先来看样本权重的更新：
{% raw %}
$$
\begin{alignedat}{}
w_{k+1i}&=\frac{w_{ki}}{Z_{k}}exp(-\alpha _{k}y_{i}G_{k}(x_{i})) \\
Z_{k}&=\sum_{i=1}^{m}w_{ki}exp(-\alpha _{k}y_{i}G_{k}(x_{i}))
\end{alignedat}
$$
{% endraw %}
则有
{% raw %}
$$
\begin{alignedat}{2}
w_{1i}&=\frac{1}{m} \\
w_{2i}&=\frac{w_{1i}}{Z_{1}}exp(-\alpha_{1} y_{i}G_{1}(x_{i})) \\
&= \frac{1}{m} \frac{1}{Z_{1}} exp(-\alpha_{1} y_{i}G_{1}(x_{i}))\\
&= \frac{1}{m} \frac{1}{Z_{1}} exp(-y_{i}f_{1}(x_{i}))\\
w_{3i}&=\frac{w_{2i}}{Z_{2}}exp(-\alpha_{2} y_{i}G_{2}(x_{i})) \\
&= \frac{1}{m} \frac{1}{Z_{1}} \frac{1}{Z_{2}} exp(-y_{i}f_{1}(x_{i}))exp(-\alpha_{2} y_{i}G_{2}(x_{i})) \\
&=\frac{1}{m} \frac{1}{Z_{1}} \frac{1}{Z_{2}} exp(-y_{i}f_{2}(x_{i}))\\
...\\
w_{ki}&=\frac{1}{m} \prod_{j=1}^{k-1}\frac{1}{Z_{j}} exp(-y_{i}f_{k-1}(x_{i}))\\
\because\\
w_{ki}^{'}&=exp(-y_{i}f_{k-1}(x_{i})) \\\\
\therefore\\
w_{ki}&=\frac{1}{m} \prod_{j=1}^{k-1}\frac{1}{Z_{j}} w_{ki}^{'}\\
w_{ki}^{'}&=m\prod_{j=1}^{k-1}Z_{j}w_{ki}\\
err^{'}&=\frac{\sum_{y_{i}\neq G_{k}(x_{i})}w_{ki}^{'}}{\sum_{i=1}^{m}w_{ki}^{'}}\\
&=\frac{\sum_{y_{i}\neq G_{k}(x_{i})}(m\prod_{j=1}^{k-1}Z_{j}w_{ki})}{\sum_{i=1}^{m}(m\prod_{j=1}^{k-1}Z_{j}w_{ki})}\\
&=\frac{m\prod_{j=1}^{k-1}Z_{j} *\sum_{y_{i}\neq G_{k}(x_{i})}w_{ki}}{m\prod_{j=1}^{k-1}Z_{j}*\sum_{i=1}^{m}w_{ki}}\\
&=\frac{\sum_{y_{i}\neq G_{k}(x_{i})}w_{ki}}{\sum_{i=1}^{m}w_{ki}}\\
&=e
\end{alignedat}
$$
{% endraw %}
其实，从
$$w_{ki}^{'}=exp(-y_{i}f_{k-1}(x_{i})) $$
也可以看出样本的分类损失直接决定了该样本的新权重。损失越大，后续权重越大

未完待续
如有错误欢迎指正

参考：
[刘建平博客-adaboost](http://www.cnblogs.com/pinard/p/6133937.html#!comments)
统计学习方法-李航
[adaboost推导](http://www.csuldw.com/2016/08/28/2016-08-28-adaboost-algorithm-theory/)
[误差限](https://www.jianshu.com/p/bfba5a91ba15)