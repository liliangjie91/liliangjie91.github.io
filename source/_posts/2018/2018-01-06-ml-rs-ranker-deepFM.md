---
title: 古早博文-推荐系统-精排-CTR-DeepFM等DNN模型
date: 2018-12-31 20:36:29
categories: 
    - 技术
    - 推荐系统
tags:
    - 机器学习
    - 推荐系统
    - 排序模型
mathjax: true
---
### 概述
关键词：特征组合
**LR：**缺乏特征组合能力，需人工做特征工程
**GBDT+LR：**特种组合能力不强，对高维的稀疏特征应对乏力
**FM：**具有较强的二阶特征组合能力，高阶特征组合应对乏力
**Wide&Deep：**较好地利用了低阶和高阶组合特征。但是wide部分依旧需要特征工程，其二阶特征组合需要人工挑选
**DeepFM ：**其实是Wide&Deep的变体，把wide部分由LR转变为FM。所以更好地利用了低阶和高阶的组合特征。而且免去了人工特种组合。

**说到底，为什么CTR领域，特征组合如此被强调？**
首先，CTR领域特征基本属于离散类别特征。
而不同的特征之间可能存在某种联系。例如：
* 1，性别男+20-30岁+喜欢篮球 可能更倾向于点击 枪战的游戏广告
* 2，饭点时间段 用户可能更容易去下载外卖类app
* 3，经典的尿布-啤酒案例

像第一种组合，可能很容易就能想出来，甚至不需要数据的支持
而第二种组合，就不太容易能想出来，但可能通过简单的数据分析而发现此规律
但像第三种组合，几乎只能通过数据发掘出来。
这就提示我们，**数据中可能存在很多难以人工发掘甚至不能通过简单数据分析而得出的有用的组合特征**这些组合特征可能是低阶的也可能是高阶的，而且高低阶组合特征都很重要。
**所以要尽可能做特征组合。**
*其实在NLP中，n-gram也属于特征组合。只不过会带来更高的特征维度。因为1，词表太大，2，词不分field。最终会导致词的embedding矩阵太大。*

### DeepFM
#### 模型  
![DeepFm](/image/image-DeepFm.png)
如上图所示：
**其实DeepFM就是把Wide&Deep模型的wide部分改为了FM。**
**黑色线**---带权重的连接
**红色线**---不带权重的连接
**蓝色线**---稀疏特征向稠密特征转换的embedding向量，**并且这个embedding会随着训练而学习更新**

* 第一层：Sparse Feature 稀疏特征，每个field下有一个onehot编码的向量，其长度等于此filed下特征类别数，共m个field。
* 第二层：m个$Dense$向量拼接而成的$mk$维向量x。
第一层中的 **每个field i**都转换(Embedding)到了k维稠密向量空间$Dense_i$，filed i中每个不同的onehot向量(具体的一个特征如$[0,0,1,0,0,0,0]$)对应着一个特定的稠密向量$Dense_{ik}$，例如对于**某个filed i**，可能有如下对应(k=4)
$$
\begin{alignedat}{2}
[1,0,0,0,0,0,0] &\Rightarrow [0.2,0.3,0.5,0.2]=Dense_{i1}\\
...\\
[0,0,0,0,0,1,0] &\Rightarrow [0.4,0.3,0.9,0.3]=Dense_{i6}\\
[0,0,0,0,0,0,1] &\Rightarrow [0.1,0.1,0.3,0.8]=Dense_{i7}\\
\end{alignedat}$$
$Dense_{i1},...Dense_{i7}$都是空间$Dense_i$里的向量
注意，这里的[1,0,0,0,0,0,0]仅代表了一个field，而非一个样本，一个样本由若干个这样的filed拼接而成。
即**这里把每一个field映射到了对应的一个k维空间里面**
* 第三层FM Layer：
一个Addition符号，接收黑色连线，代表了FM的一阶特征部分$$\sum w_ix_i$$
后续的若干个Inner Product接收红色连线，代表了二阶特征组合部分$^1$(见探究部分) 。对于每一个样本$$\sum \sum <v_i·v_j>x_ix_j=\sum \sum <Dense_{ia}·Dense_{ja}>$$ **对于field内互斥的onehot编码，对于一个特定样本，$v_i=Dense_{ia},v_j=Dense_{ja}$**
注意这里，两个$\sum$部分是在第四层Output Units完成的，本层只交叉出该样本的组合特征。
* 第三层Hidden Layer：简单的全连接。
输入是m个$Dense$向量的拼接$a^0=x$，维度是$mk$
后续是$a^l=W^{l-1}a^{l-1}+b^l$
* 第四层 Output Units。接收第三层的输出：
$$\sum w_ix_i +\sum \sum <v_i·v_j>x_ix_j+W^la^l $$
最后的输出是
$$P(Y=1|x)=\sigma(\sum w_ix_i +\sum \sum <v_i·v_j>x_ix_j+W^la^l)$$

#### FM层的探究
在[FM]([https://www.jianshu.com/p/fce17c3ac3a2](https://www.jianshu.com/p/fce17c3ac3a2)
)中
对于onehot编码来说，
$x_i$特征对应着向量$[0,0...1...0...0]$其中第i位=1，其他=0；  

$x_i$特征对应着向量$[0,0...0...1...0]$其中第j位=1，其他=0；

所以$x_i$的隐向量=$v_i$等价于$[0,0...1...0...0]$被编码成$v_i$：
$$v_i=transform([0,0...1...0...0])$$
而$x_i * x_j$表示两特征交叉，其权重$w_{ij}$则被FM表示成了两隐向量的內积$<v_i·v_j>$
则$$<v_i·v_j>x_i*x_j=transform([0,0...1...0...0])·transform([0,0...0...1...0])$$
注意式子
$$\sum_{i=1}^{n-1}\sum_{j=i+1}^{n}<v_i·v_j>x_ix_j \tag{4.1}$$
虽然看似有$o(n^2)$的复杂度，但实际上$x_ix_j$绝大多数时候都=0，这表示对于每一个样本，式(4.1)计算量远达不到$o(n^2)$。对于一个特定的样本，式(4.1)退化成
$$\sum_{i=1}^{\overline{n} -1}\sum_{j=i+1}^{\overline{n}}<v_i·v_j>x_ix_j \tag{4.2}$$
$\overline{n}$表示该样本的稀疏特征中1的数量。
即
$$
\begin{alignedat}{2}
\sum_{i=1}^{\overline{n} -1}\sum_{j=i+1}^{\overline{n}}<v_i·v_j>x_ix_j = \sum_{i=1}^{\overline{n} -1}\sum_{j=i+1}^{\overline{n}}transform([0,0...1...0...0])·transform([0,0...0...1...0])
\end{alignedat}$$

在DeepFM的FM层中，对于特定的样本，二阶特征组合被表示成了
$$\sum\sum<Dense_{ia}·Dense_{ja}>$$
$Dense_{ia},Dense_{ja}$表示样本a分别在filed i和j的具体Dense特征。这其实就是FM的隐向量$v_i,v_j$
$$\sum\sum<Dense_{ia}·Dense_{ja}> =\sum\sum<v_i·v_j> $$
而要使这个等式成立，需要满足如下条件：
* 1，所有Dense向量维度相等都为k。这个显而易见。
* 2，特征的交叉限于field之间，而非filed之内。但FM中没有field概念，FM特征交叉是每一个特征都会交叉，会比DeepFM的交叉特征多很多。
* 3，稀疏编码应该是One-Hot(向量中只有一个1，如性别，分桶后的年龄)的。而非Multi-Hot(一个向量中有多个1，如安装的app，爱好)。
FM无论是onehot还是multihot编码，都会构建出所有二阶交叉的特征(例如性别和某一个app的交叉)。
DeepFM只有在onehot编码下才能构建出所有二阶交叉特征。当每个field只用onehot编码时，就不会存在field内的特征交叉(就像不存在性别男x性别女的交叉一样)。但如果有的field是multihot编码的，即一个样本在一个field中同时存在多个1，即应该使用多个隐向量$v_i$，那么按照DeepFM或wide&deep的网络结构，就必须把这多个隐向量$v_i$(或称$Dense_{ia}$)合而为一(例如求和)。这样就无法做该filed内的特征组合了(例如无法完成“安装了微信x安装了支付宝”这样的特征交叉，而这样的交叉显然是很有意义的)。但是field间的特征组合还是可以做的，因为根据乘法分配律$y\sum x_i=\sum yx_i$，field间依旧可以做二阶特征交叉。

如果要解决multihot的问题，就需要把每一个特征化为一个field。但这样会带来一个问题，就是DenseEmbedding层维度太高(相当于稀疏编码的k倍！)，造成神经网络参数过多影响速度，所以DeepFM也算是一种折中(但结果还是有提高的)
在DeepFM的论文中，特征只使用了onehot编码，巧妙地规避了这个问题。

而反观wide&deep模型，是使用了multihot编码的(user installed app，impression app)，在转化为dense embedding时可能用了求和或平均然后送入Deep模块；但同时模型把稀疏的multihot编码做了人工特征挑选和交叉后送入了Wide模块。所以可以认为wide&deep模型反而比DeepFM模型多做了field内特征交叉的工作(但是是非自动化的，亦非全部field的)。所以在multihot编码问题上，DeepFM还有提升空间。例如对于multihot特征，单独使用一个FM做field内交叉。

### 其它FM与神经网络组合模型
![FNNPNN](/image/image-FNNPNN.png)
#### FNN
使用FM初始化DenseEmbedding层，然后直接输入全连接网络。
需要预训练。在神经网络训练后，高阶组合特征得到了较好地训练，但可能因而使得低阶特征被忽略

#### PNN
从稀疏层到DenseEmbedding层是类似DeepFM的随机初始化+后续训练。只不过在DenseEmbedding层到隐层之间，又多了一层Product层。这个Product层用于组合DenseEmbedding层中不同field的特征。怎么组合呢？
看上图中，在Product层，有两部分组成：
* 灰底灰度点部分表示DenseEmbedding的直接拼接
* 黄点部分则表示DenseEmbedding不同field特征的两两组合。而这里的两两组合又有很多种形式：
  * 第一种，是求內积，就像DeepFM的FM部分一样，只不过其结果送入了隐层 。模型叫做IPNN
  * 第二种，是求外积，这样两个field特征交叉得到一个矩阵而非实数。而这两两获得的多个矩阵，如果通过拼接来组合那就会带来计算量的暴增，所以为了降低计算量，会把这多个矩阵累加起来。
#### NFM
像PNN，在DenseEmbedding层和隐层之间加一层FM层做特征组合。其实比PNN简单
#### AFM
引入注意力机制
待续

### 参考
DeepFM: A Factorization-Machine based Neural Network for CTR Prediction
[https://cloud.tencent.com/developer/article/1164785](https://cloud.tencent.com/developer/article/1164785)
