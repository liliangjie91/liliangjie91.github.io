---
title: 古早博文-推荐系统-精排-CTR-Wide&Deep模型
date: 2018-12-30 20:34:33
categories: 
    - 技术
    - 推荐系统
tags:
    - 机器学习
    - 推荐系统
    - 排序模型
mathjax: true
---
#### 模型
![Wide模型-Deep模型-Wide&Deep模型](/image/image-wide-deep1.png)
上图讲得十分清楚：
第一层(最下层)黄点和灰点，表示稀疏离散特征。
第二层表示对稀疏离散特征embedding后的稠密特征向量。
第三层就是深度模型，这里使用了ReLu激活函数
第四层就是输出单元，sigmoid激活也可以认为是LR
* 1，最左边的Wide模型其实就是LR模型。最右面Deep模型其实就是深度模型了。中间是两者结合的Wide&Deep模型，其输出单元接收的是左右两部分输出的拼接。
* 2，对于两部分模型(Wide,Deep)的输入，看下图
![Wide&Deep具体应用模型](/image/image-wide-deep2.png)

这是google paly 商店的推荐应用，wide模型和deep模型接受了不同的特征。
deep模型直接接收连续特征和embedding后的离散特征。其输出的即
$a^{l}=W^{l}a^{l-1}+b^{l}$
wide模型只接受了部分离散特征：user installed app即用户安装的app，impression app即用户看到的被展示的app(看到了但没下载)，以及这两部分特征的交叉。其输出即
$[x,\phi(x)]$其中$\phi(x)$ 是交叉特征。
#### 优化
最终模型输出是
$$P(Y=1|x)=\sigma(W_{wide}[x,\phi(x)]+W_{deep}a^l+b)$$
每一批数据，模型同时优化$W_{wide},W_{deep}$只是在优化算法上
$W_{wide}$ 使用**Follow-the-regularized-leader (FTRL)**+**L1正则** [FTRL优化算法](https://blog.csdn.net/china1000/article/details/51176654)
$W_{deep}$使用**AdaGrad**
注意这里的同时优化，google论文中叫joint training(联合训练)。并非ensemble(集成学习)
这是因为对于集成学习，每个子学习器的训练都是独立的，两个子学习器参数不会一起训练。
而联合训练指$W_{wide},W_{deep}$参数是同时更新的。
#### Embedding 层
* 为什么需要做embedding？
超高维度的稀疏输入输入网络，将带来更高维度的参数矩阵，这会带来更大的计算压力。所以神经网络更善于处理稠密的实值输入。所以，需要对稀疏的离散特征做embedding
* 怎么做embedding？
  * 1，离线提前做embedding，例如对于词的嵌入可以使用Word2vec对词做嵌入。也可利用FM先学习好稀疏特征的隐向量。
  * 2，随机初始化。之后跟着模型参数一起训练。其实1中无论是word2vec还是FM，也是一开始随机初始化，然后训练学习而来。这是最常用的方法。

在wide&deep中，embedding是随机生成的，并在接下来的训练中更新
>The embedding values are initialized randomly, and are trained along with all other model parameters to minimize the training loss.

通常会用
```
tf.feature_column.embedding_column(categorical_column,
                     dimension,
                     combiner='mean',
                     initializer=None,
                     ckpt_to_load_from=None,
                     tensor_name_in_ckpt=None,
                     max_norm=None,
                     trainable=True)
```
注意参数`trainable=True`即默认会继续训练这个embedding
而当设置`trainable=False`时，一般配合已经训练好的embedding
#### 讨论
* wide与deep分别代表了什么？
wide是简单的线性模型，他会记住训练数据中**已经出现**的模式，并赋予权重。这代表了**记忆**
deep是深度的复杂模型，会在一层层的网络中计算出训练数据中**未出现**的模式的权重。这代表了**泛化**
这里的模式，可以简单理解为特征组合。
>Wide侧就是普通LR，一般根据人工先验知识，将一些简单、明显的特征交叉，喂入Wide侧，让Wide侧能够记住这些规则。
>Deep侧就是DNN，通过embedding的方式将categorical/id特征映射成稠密向量，让DNN学习到这些特征之间的深层交叉，以增强扩展能力。
* 但其实deep模型本身也会记住已出现的模式并进行训练吧？相当于低阶特征也可以得到有效利用，为什么还要加上wide模型呢？
可能原因：deep模型可解释性不强。wide模型可解释性强。通过wide模型可以挑选出权重较高的低阶特征。同时，对低阶特征另外单独建模，也是很有可能提高精度的。

* 其他

>https://zhuanlan.zhihu.com/p/47293765
相比于实数型特征，稀疏的类别/ID类特征，才是推荐、搜索领域的“一等公民”，被研究得更多。即使有一些实数值特征，比如历史曝光次数、点击次数、CTR之类的，也往往通过bucket的方式，变成categorical特征，才喂进模型。

>推荐、搜索喜欢稀疏的类别/ID类特征，可能有三方面的原因：
https://zhuanlan.zhihu.com/p/47293765
1,LR, DNN在底层还是一个线性模型，但是现实生活中，标签y与特征x之间较少存在线性关系，而往往是分段的。以“点击率-历史曝光次数”之间的关系为例，之前曝光过1、2次的时候，“点击率-历史曝光次数”之间一般是正相关的，再多曝光1、2次，用户由于好奇，没准就点击了；但是，如果已经曝光过8、9次了，由于用户已经失去了新鲜感，越多曝光，用户越不可能再点，这时“点击率~历史曝光次数”就表现出负相关性。因此，categorical特征相比于numeric特征，更加符合现实场景。
2,推荐、搜索一般都是基于用户、商品的标签画像系统，而标签天生就是categorical的
3,稀疏的类别/ID类特征，可以稀疏地存储、传输、运算，提升运算效率。

####参考
[https://blog.csdn.net/google19890102/article/details/78171283](https://blog.csdn.net/google19890102/article/details/78171283)
[https://zhuanlan.zhihu.com/p/47293765](https://zhuanlan.zhihu.com/p/47293765)
Wide & Deep Learning for Recommender Systems - - Google
[FTRL优化算法](https://blog.csdn.net/china1000/article/details/51176654)
[tensorflow_wide&deep.md](https://github.com/tensorflow/tensorflow/blob/752dcb61ef7a8fd6555909dc37c1f2a2e5792227/tensorflow/docs_src/tutorials/wide_and_deep.md)
[wide_deep的一个实现](https://github.com/tensorflow/models/tree/master/official/wide_deep)
