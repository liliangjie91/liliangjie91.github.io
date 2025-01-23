---
title: 大语言模型演进史-06-BERT
mathjax: true
date: 2025-01-21 21:20:21
categories:
    - 技术
    - AI大模型
tags: [大模型演进, AI模型, NLP, transformer, BERT]
---

# 模型结构

<img src="/image/2025/image-bert1.png" width=700 height=300 />

BERT整体宏观结构和GPT一样很简单，就是N个Transformer块的堆叠在加上输出层。    
对于BERT而言，其使用的是Transformer的Encoder模块。即使用self-attention。
<!--more-->

# 数据
预训练数据：  
BooksCorpus (800M words) + English Wikipedia (2,500M words) = 3.3B

# 模型训练
因为要和GPT一样使用预训练+微调的方式。  
所以在模型设计上，就要**考虑如何兼容下游任务/或者微调时如何让不同任务输入调整为和预训练相同的格式**。
为了兼容不同的下游任务，在预训练时，作者设计了两种预测任务：
- 预测掩词（Masked LM，MLM）
- 下一句预测（Next Sentence Prediction，NSP）
## 预训练
### 预测掩词（Masked LM ，MLM）
为了训练语言模型，必然要有预测任务。  
但是模型使用的是Transformer的**Encoder**，所以不能像GPT那样使用“Next Word Predict”的方法训练。  
因为Encoder天然可以“看到”后面的词，如果要使用masked attention，那encoder就变成decoder了，所以此路不通。  
如果使用像word2vec中skip-gram或CBOW模型，则压根无法发挥模型的特性。  
所以作者选择**Masked LM模型**，保证输入可以是序列。  
**即利用了encoder“双向编码”的特性，又避免了encoder“看到”预测词导致训练失效。**
一句话概括MLM：把一句话中部分词盖住，让模型预测被盖住的词（完形填空）

#### MLM举例
原始输入是: `my dog is hairy`  
随机选择这句话中15%的词盖住，例如选中了hairy  
则: `input = “my dog is [MASK]” ; label = “hairy”`
因为选中了输入序列中第4个词/token，所以预测也是选择线性层第4个输出向量，并经过Softmax层做预测。

#### MLM具体细节
- 对每个输入序列seq，都要对其中随机15%的词做预测，而相对于GPT，其实是对每一个词做预测，即每一批次的训练次数有差异。
- 即便在对这15%的词做预测时，也不是全部将其替换为`[MASK]`这个Token，而是遵循下面的比例：
{% raw %}
$$
\text{my dog is hairy}
\begin{cases}
  \text{替换为[MASK]： my dog is [MASK] } & \text{ 80\% } \\
  \text{替换为其他词： my dog is apple } & \text{ 10\% } \\
  \text{不做替换： my dog is hairy } & \text{ 10\% } 
\end{cases}
$$
{% endraw %}  
**这样引出三个问题：**
1. 为什么不都使用[MASK]？  
作者解释，“[MASK]”这个Token其实是仅存在于预训练过程的，在微调时是不使用这个token的，这导致预训练和微调mismatch。
同时如果替换为其他词或不做替换，那么对于encoder而言，他不知道哪个词需要被预测，所以被迫学习上下文信息`so it is forced to keep a distributional contextual representation of every input token`  
从下表可以看出，
    - 如果全部使用MASK，在命名实体识别任务NER中效果会变差较多。
    - 如果仅使用MASK和随机词替换，效果会有一点点的下降。
    - 仅使用MASK和“不替换”，效果竟比上面要好。
<img src="/image/2025/image-bert2.png" width=300 height=200 />    
但整体上，各种变化引起的效果差异都是很微小的。

2. 为什么要替换成其他词？  
要提高模型的纠错/抗干扰能力？
3. 为什么有一定概率不做替换？  
作者解释：The purpose of this is to bias the representation towards the actual observed word.
无论是MASK还是随机词替换，其实都是不利于self-Attention过程中其他词的向量计算的，相当于引入了偏移。
而“不做替换”相当于纠正这种偏移。

### 下一句预测（Next Sentence Prediction ，NSP）
仅有MLM训练方式是不够的，因为很多NLP任务是判断**两段输入**的关系，例如QA（输入问题+文本输出答案）或NLI（例如文本蕴含等任务）
所以模型需要能够接受2句输入。以此，作者设计了下一句预测任务，用于预测后一句是否是前一句话的延续。
实验证明，如果不做该任务，对应类的任务效果下降较大。
#### 数据举例
`Input = [CLS] the man went to [MASK] store [SEP] he bought a gallon [MASK] milk [SEP]`  
`Label = IsNext`  
`Input = [CLS] the man [MASK] to the store [SEP] penguin [MASK] are flight ##less birds [SEP]`  
`Label = NotNext`  
正负样本比例1:1  

### 训练过程
<img src="/image/2025/image-bert3.png" width=700 height=400 />  

预训练是NSP和MLM的结合。  
输入的两句话以`[CLS]`开始，以`[SEP]`分隔。  
在预测时，`[CLS]`对应的输出位置预测NSP的label；  
`[MASK]`对应的输出位置预测MLM的词。

## 微调
<img src="/image/2025/image-bert4.png" width=700 height=600 />  
针对不同的下游任务，设置不同的输出结构。
在此可以看到`[CLS]`这个Token的作用，更像是对整个输出序列的总结，类似Doc2vec。

# 模型参数
<style>
  table {
    width: 60%;
    font-size: 12px;
  }
  th, td {
    padding: 2px;
    text-align: center;
  }
</style>

|模型|Eecoder层|注意力头|维度|前馈维度|参数量|词表|
|--|--|--|--|--|--|--|
|$BERT_{base}$|12|12|768|768*4|0.11B=1.1亿|3w-WordPiece|
|$BERT_{large}$|24|16|1024|1024*4|0.34B=3.4亿|3w-WordPiece|

**预训练时：**
|优化器|激活函数|lr|seqlen|batch|epoch|dropout|L2正则
|--|--|--|--|--|--|--|--|
|Adam|GELU|max=1e-4|512(128-90%)|256|40|0.1|0.01|

其中学习率是线性增到max（10000step）再linear衰减到0

**微调时：**（不同任务使用了不同参数）
|lr|batch|epoch|
|--|--|--|
|5,3,2e-5|16,32|2,3,3|

# 讨论
## 论文出发点
- **论文第一句话是介绍什么是BERT：**  
We introduce a new language representa-tion model called BERT, which stands for  
**B**idirectional **E**ncoder **R**epresentations from **T**ransformers.  
- **论文第二句指明了自己对标的模型：ELMo和GPT**  
Unlike recent language repre-sentation models (**Peters et al.2018a; Rad-ford et al.2018**), BERT is designed to pre-train deep **bidirectional** representations from unlabeled text by **jointly** conditioning on both left and right context in all layers.  
同时指明了其相比对标模型的优势：
    1. bidirectional/双向语言模型（对比GPT的单向）
    2. jointly/双向联合学习（对比EMLo的两个方向独立训练模型之后拼接）

这样很容易看出BERT的**灵感来源/设计目的**：组合当前NLP版图中还未被利用的“工具”，结合预训练+微调的思想。

## BERT对比GPT
从预训练和微调可以看出，BERT并不适合**文本生成**任务。  
因为首先在训练时，BERT就不是为了文本生成而设计的，他没有设计“自回归”的模块。  
但BERT侧重的是文本理解，包括对**词的理解（MLM）和对句子的理解（NSP）**。
- GPT相当于把所有问题都交给了最后一个Transformer模块的最后一个token。
- BERT则有选择的把问题交给最后一个Transformer模块的第一个token（`[CLS]`）和后续所有位置

论文中也给出了对于BERT作为特征（Feature base）的下游应用效果，整体稍弱于基于微调的方法。但相对而言，其词向量也已经是非常好的了。  
所以在很多实际应用中，大家都选择使用BERT的词向量。这对BERT的推广也起到很大作用。

仅对比BERT和GPT1，BERT明显是更优秀的，无论是具体效果还是后续的工业应用。所以当时基于BERT的后续研究和应用非常多。

## 预训练思路
预训练指先使用大规模数据训练基础模型。后续如何利用该模型，又分为两种：基于特征，基于微调。  
- EMLo是**基于特征**的，即训练好基础模型后，使用其词向量，组合之后应用到下游模型。
- GPT是**基于微调**的，即训练好基础模型后，不迁移到其他模型，只稍微调整模型小部分结构，直接处理下游任务。

由此可看出，基于微调的模型，更统一，使用更方便。

### 相关工作介绍
1. 无监督特征训练方法
像word2vec，doc2vec等以及最后的ELMo模型，侧重于向量的使用
2. 无监督微调方法
像GPT这种
3. 有监督的迁移学习
指明预训练+微调的有效性，举例CV领域的imageNet

