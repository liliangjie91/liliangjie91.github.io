---
title: 大语言模型演进史-07-GPT2-3
mathjax: true
date: 2025-01-23 21:20:21
categories:
    - 技术
    - AI大模型
tags: [大模型演进, AI模型, NLP, transformer, GPT]
---

# 模型结构

<div style="display: flex; justify-content: space-between;">
  <img src="/image/2025/image-gpt2-more.png" style="width: 74%;" alt="Image 2">
  <img src="/image/2025/image-gpt2.png"  style="width: 25%;" alt="Image 1">
</div>
<!--more-->
GPT2,3的模型结构和GPT1大致相同，依旧是Decoder-Only的模型，差异都在图中标出，整体是在向更大发展。

# GPT-2

## 数据
作者希望使用高质量的数据集，于是自己制作了WebText数据集，
主要来自Reddit上的质量较高的网页链接4500万个对其处理后整理成800万个文档（GPT1是7000部小说），共计40GB大小(GPT1是5GB)。
- 去除维基百科数据
为了防止模型在这些数据中学到测试集，而导致效果的说服力不足（或者是想和BERT区分开？）。

## 模型训练
### 预训练
为了实现Zero-Shot这一目标，模型的输入结构要做改变：
GPT1中，为了应对各种NLP任务，微调时会加入功能性Token如`开始符（Start）、分隔符（Delim）、结束符（Extract）`  
而要实现Zero-Shot就不能微调，第一个问题就是模型如何知道它当前要处理的是什么任务呢？  
所以，让**模型知道自己要处理什么任务**这个任务就落在了预训练头上。
这就需要对预训练数据做处理：
例如针对翻译任务，就需要在原数据基础上，在前面加上“translate english to chinese”，一条数据就是如下格式：
`translate english to chinese, [englist text], [chinese text] `
针对问答任务：
`answer the question, [document], [question], [answer]`
相当于针对每一条训练数据，都标注了任务类型。只要这样的数据够多，模型就能学到**以哪些句子开头的输入对应哪些任务**。

## 模型参数
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
|modle|Decoder层|注意力头|维度|前馈维度|参数量|词表|
|--|--|--|--|--|--|--|
|GPT1|12|12|768|768*4|0.1B=1亿|4w-BPE|
|GPT2|48|12|1600|1600*4|1.5B=15亿|5w-BPE|

**预训练时：**
|modle|优化器|激活函数|lr|seqlen|batch|epoch|dropout|L2正则
|--|--|--|--|--|--|--|--|--|
|GPT1|Adam|GELU|max=2.5e-4|512|64|100|0.1|0.01|
|GPT2|-|-|-|1024|512|100|0.1|0.01|

其中学习率是线性增到max（2000step）再cosine衰减到0

## 讨论
### 论文出发点
GPT1可以说是被BERT碾压，那么GPT如果还要做下去，就要比BERT效果提高一些。
但是单纯增大GPT的规模和数据，却很难打过或远超同规模BERT。
这种情况下，作者需要有其他的创新点，否则论文就没有任何新颖的地方了。
本文就提出了**另一个赛道zero-shot**：属于“我长跑跑不过你，有本事来比比跨栏“。
所以关键就在于作者要说服大家这个新赛道是有意义的，是一个更宏大赛道。

### Zero-Shot
GPT1，BERT出现之后，NLP领域预训练+微调的模型已经被证实有效。那如何更进一步呢？
**那就是预训练完的模型可以直接处理各种NLP任务，即不再微调。**  
这种模型预训练完之后直接输入问题模型就能给出答案的方式，就是**Zero-Shot**
很显然，**Zero-Shot**模型是更易用的，更像是人工智能的。

### 贡献
提出了**Zero-Shot**
验证了仅使用预训练就可以完成下游NLP任务。
提出了使用Prompt来激活模型的zero-shot能力
提出随着模型规模增大，其能力仍有上升空间


# GPT-3

## 数据

<img src="/image/2025/image-gpt3-data.png" width=600 height=250 />
与GPT2不同，GPT3对数据的态度是多多益善，还使用了gpt2所“嫌弃”的Common Crawl数据集，以及gpt2所剔除的维基百科数据集。
当然，GPT3对这些数据进行了大量清洗，过滤，去重等操作
45T的原始数据，清洗后为**570G**，是GPT2的16倍，GPT1的110倍。总计500B个训练Token。

## 模型参数
总参数量=**175B 是GPT2的110倍，GPT1的1700倍**
值得注意的是，更大的模型一般会使用更大的batch size，但只需要更小的learning rate。
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
|modle|Decoder层|注意力头|维度d|前馈维度|参数量|词表|
|--|--|--|--|--|--|--|
|GPT1|12|12|768|d*4|0.1B=1亿|4w-BPE|
|GPT2|48|12|1600|d*4|1.5B=15亿|5w-BPE|
|GPT3|96|96|12288|d*4|175B=1750亿|5w-BPE|

**预训练时：**
|modle|优化器|激活函数|lr|seqlen|batch|epoch|dropout|L2正则
|--|--|--|--|--|--|--|--|--|
|GPT1|Adam|GELU|max=2.5e-4|512|64|100|0.1|0.01|
|GPT2|Adam|GELU|-|1024|512|100|0.1|0.01|
|GPT3|Adam|GELU|0.6e-4|2048|3.2M|1|0.1|0.01|


## 讨论
### 论文出发点
有了GPT2的铺垫，GPT3要做的就是证明GPT2的路是对的（解决GPT-2的有效性问题）

### few-shot
可能zero-shot这一步迈的太大，论文着重讲了few-shot。
看一下few-shot，one-shot，zero-shot和fine-tune的区别
<style>
  table {
    width: 80%;
    font-size: 12px;
  }
  th, td {
    padding: 2px;
    text-align: center;
  }
</style>
|类型|预训练后是否还更新权重|整体流程|样例个数|
|--|--|--|--|
|fine-tune|是|预训练+有监督训练|-|
|few-shot|否|预训练+任务描述+样例+问题/prompt|10-100|
|one-shot|否|预训练+任务描述+样例+问题/prompt|1|
|zero-shot|否|预训练+任务描述+问题/prompt|0|

Few-shot又叫in-context learning，即，模型在输入的样例中学会了如何处理当前问题。

下图展示4中方法的具体不同：

<img src="/image/2025/image-gpt3.png" width=700 height=700 />

但实际上作者并没有放弃zero-shot。论文中few-shot效果更好，这也很容易理解。
但few-shot也仅仅是一个发展阶段，最终的目标依旧是zero-shot。
GPT3是GPT2中“舍弃微调”这一思想的延续。
论文用大量的实验夯实了GPT2的观点：**大参数大数据集预训练+prompt可以替代微调**
人们使用NLP模型的门槛又降低了（虽然不用微调，但是模型体积却变大了，也为后来的大模型API埋下伏笔）
或者说一个模型解决所有NLP问题的可能性又变大了。

### 走向zero-shot
GPT接下来的问题就是把few-shot也“训练”进模型，从而使模型获得zero-shot的能力，
这也是后续instructGPT以及ChatGPT所做的事。
