---
title: 古早博文-机器学习-泛化误差与偏差bias与方差Variance
date: 2017-12-31 19:58:26
categories: 
    - 技术
    - 机器学习
tags:
    - 机器学习
mathjax: true
---
我们之所以需要拟合，就是因为我们**难以甚至无法获得全部真实数据**
如果我们可以获取完备的真实数据集，那么我们压根就不需要做拟合了，我们只要查询就好了。
所以，我们能获取的数据，以及能够用于训练的数据，只是真实数据的一部分，而且，我们也是假设，训练数据分布与真实数据独立同分布，所以训练数据越多，可以认为越接近真实分布。


我们用训练误差$Err_{train}$表示模型在训练集上表现好坏
用泛化误差$Err_{test}$表示模型在测试集上表现好坏

实际上，训练数据分布与真实数据分布是有一定偏差的，而且，数据本身也存在噪声。
这就暗示我们，如果我们只用训练数据去完美地拟合/训练一个模型M1即$Err_{train-M1}=0$，但它在实际测试数据上很可能是不完美甚至是很差的即$Err_{test-M1}>>0$，这就叫**过拟合**。
而如果我们连训练数据都拟合地很差$Err_{train}>>0$，那在实际数据上一定也很差$Err_{test}>>0$，这就叫**欠拟合**。


#### 泛化误差
所谓泛化误差，即训练好的模型使用测试数据评测时的误差。
我们的根本目的是降低泛化误差，因为训练一个模型，其**根本目的是用于预测未知数据而不是训练数据。**
对于一个真实的训练任务,其样本值往往是由可解释的规律部分和不可解释的噪音组成的即
$$Y=f(x)+e$$
e可以认为是难以通过模型训练的噪音，我们往往会忽略(因为很小)
所以我们要拟合的部分是f(x),而不是去拟合Y(如果忽略e，那就是拟合Y)
用训练数据D训练的模型称之为$\widehat f(x)$ 注意此处是戴帽子的f(x)
当我们使用相同的算法，但使用不同的训练数据D时就会得到多个$\widehat f(x)$则
$$E(\widehat f(x))$$
代表了这个模型的期望，即使用某一算法训练模型所能得到的稳定的平均水平。
##### 方差variance
$$var=E\bigg[(E(\widehat f(x))-\widehat f(x))^2\bigg]$$
代表了这个模型/算法的稳定性。我们称之为方差。
如果方差很大，则代表相同算法在不同训练数据上会得到相差很大的结果，这往往表示模型训练过拟合，不同的$\widehat f(x)$拟合曲线相差很大，这样就会导致对同一个测试样本，结果相差大。这表示数据的变化会给模型带来很大的扰动，就像打靶一样，射点不集中
##### 偏差 bias
而
$$bias^2 = (E(\widehat f(x))-f(x))^2$$
此称为偏差bias。注意这里为何不再加一个期望符号E了呢，因为括号内两者都已经是定值了，而不是离散值。如果偏差很大，即这个此模型的平均水平与真实值相差太大，简单来说就是结果整体跑偏。就像打靶一样，射点整体偏离靶心。


##### 泛化误差：
$$Err(x)=Err(\widehat f,f)+Err(Y,f)$$ 
对于泛化误差，是由**模型的损失**(这部分可以通过改善模型来减小)再加上**不可解释的噪声**(这是单纯数据的问题)带来的损失组成的。
当使用MSE作为损失函数的时候，有
那么有
$$
\begin{alignedat}{}
Err(x)&=Err(\widehat f,f)+Err(Y,f)\\
&=E\bigg[(f-\widehat f)^2\bigg] +E\bigg[(Y-f)^2\bigg] \\
&=E\bigg[((f-E(\widehat f))+(E(\widehat f)-\widehat f))^2\bigg]+\sigma_e^2\\
&=E\bigg[(f-E(\widehat f))^2+(E(\widehat f)-\widehat f)^2+2(f-E(\widehat f))(E(\widehat f)-\widehat f)\bigg]+\sigma_e^2\\
&=E[(f-E(\widehat f))^2]+E[(E(\widehat f)-\widehat f)^2]+E[2(f-E(\widehat f))(E(\widehat f)-\widehat f)]+\sigma_e^2\\
\end{alignedat}
$$
注意第三项，$(f-E(\widehat f))$是一个固定值
所以第三项
$$
\begin{alignedat}{2}
&=E[2(f-E(\widehat f))(E(\widehat f)-\widehat f)]\\
&=2(f-E(\widehat f))E(E(\widehat f)-\widehat f)\\
&=2(f-E(\widehat f))[(E(\widehat f)-E(\widehat f)]\\
&=0
\end{alignedat}
$$
所以
$$
\begin{alignedat}{2}
Err(x)&=E[(f-E(\widehat f))^2]+E[(E(\widehat f)-\widehat f)^2]+\sigma_e^2\\
&=(f-E(\widehat f))^2 + var + \sigma_e^2\\
&=bias^2 + var + \sigma_e^2
\end{alignedat}
$$
即泛化误差由偏差，方差和不可解释的噪音组成。
我们能控制的就是偏差和方差，尽可能减少他们
也能看出，过拟合与bias和var有密切关系：

| 拟合程度 | 模型复杂度 | bias | var | error | 表现 | 改善 |
| --- | --- | --- | --- | --- | --- | --- |
| 欠拟合 | 低 | 高 |  | 高 | 预测不准 | 提高模型复杂度，增加迭代，boosting，减小正则参数 |
| 过拟合 | 高 |  | 高 | 高 | 敏感易受扰动 | 降低模型复杂度，增加训练集数据，特征筛选，提高正则参数，bagging |
| 好拟合 | 中 | 低 | 低 | 低 | 准而稳 |  |




参考
[csdn1](https://blog.csdn.net/ChenVast/article/details/81385018)
[blog1](https://towardsdatascience.com/understanding-the-bias-variance-tradeoff-165e6942b229)
PRML
[bagging & var](https://zhuanlan.zhihu.com/p/36822575)