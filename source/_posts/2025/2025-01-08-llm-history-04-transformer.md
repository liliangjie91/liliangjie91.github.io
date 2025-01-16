---
title: 大语言模型演进史-04-transformer
mathjax: true
date: 2025-01-09 22:23:17
categories:
    - 技术
    - AI大模型
tags: [大模型演进, AI模型, NLP, transformer, seq2seq]
---

# 模型结构

<div style="display: flex; justify-content: space-between;">
  <img src="/image/2025/image-transformer1.png" width=400 height=500 style="width: 39%;" alt="Image 2">
  <img src="/image/2025/image-transformer-02.png" width=500 height=500 style="width: 59%;" alt="Image 1">
</div>

图1是Transformer的整体结构，宏观上是一个**编码器-解码器结构**。
- 左侧Encoder由$N$个编码器单元组成，每个编码器单元由**多头注意力(Multi-Head Self Attention)**和**前馈网络(Feed Forward)**以及**Layer-Norm**组成。
- 右侧Decoder由$N$个解码器单元组成，每个解码器单元由**掩码多头注意力(Masked Multi-Head Attention)**，**多头注意力(Masked Multi-Head Cross Attention)**和**前馈网络(Feed Forward)**以及**Layer-Norm**组成。
- 以及右侧最上面的线性+softmax层  

其核心就是**多头注意力(Multi-Head Attention)模块**
## 多头注意力
图1中，Encoder和Decoder的输入接入的就是多头注意力模块（MHA）。其具体结构如图2所示。
- 针对每一个“头”：输入$X$经过三个投影矩阵分别得到$Q,K,V$矩阵。三个矩阵经过运算得到输出。具体如图2左图-Scaled Dot-Product Attention
- 每个头的输出拼接起来，经过一个线性层。具体如图2右图-Multi-Head Attention

### Scaled Dot-Product Attention
最小的self-attention单元，即图2左图。输出即为

$$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_{k}}})V$$
- 除以$\sqrt{d_{k}}$，是因为$QK$相乘之后值会放大，要将其缩回到0-1  
- $shape(Q,K,V)=(128,64)$--假设每句128个token

### Multi-Head Attention 
输出是多个小单元输出的拼接concat 注意公式第一行的QKV与上面公式的不同
{% raw %}
$$
\begin{alignedat}{2}
MultiHead(X) &=Concat(head_{1},...,head_{n})W^{o}\\
head_{i} &=Attention(QW^{Q}_{i},KW^{K}_{i},VW^{V}_{i})\\
\end{alignedat}$$
{% endraw %}
- 其中$W^{Q,K,V}_{i}$即图2右图的线性变换Linear部分，$shape=(512,64)$  
- $W^{o}$是对拼接后的向量做一次转换$shape=(64 * 8,512)=(512,512)$

“Multi-Head Attention“结构下面有一个三分叉的箭头，即V，K，Q三个矩阵。由于是一根线的三分叉，所以初始V，K，Q是相同的。
但图2右图V，K，Q分别经过了Linear变换而变得不同。所以图2左图的Q，K，V矩阵就是不同的。
