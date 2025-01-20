---
title: 大语言模型演进史-04-Transformer
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
<!-- more -->
## 多头注意力层
图1中，Encoder和Decoder的输入接入的就是多头注意力模块（MHA）。其具体结构如图2所示。
- 针对每一个“头”：输入$X$经过三个投影矩阵分别得到$Q,K,V$矩阵。三个矩阵经过运算得到输出。具体如图2左图-Scaled Dot-Product Attention
- 每个头的输出拼接起来，经过一个线性层。具体如图2右图-Multi-Head Attention

### 1. Scaled Dot-Product Attention
最小的self-attention单元，即图2左图。输出即为

$$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_{k}}})V$$
- 除以$\sqrt{d_{k}}$，是因为$QK$相乘之后值会放大，要将其缩回到0-1  
- $shape(Q,K,V)=(128,64)$--假设每句128个token

### 2. Multi-Head Attention 
输出是多个小单元输出的拼接concat  
{% raw %}
$$\begin{alignedat}{2}
MultiHead(X) &=Concat(head_{1},...,head_{n})W^{o}\\
head_{i} &=Attention(XW^{Q}_{i},XW^{K}_{i},XW^{V}_{i})\\
\end{alignedat}$$
{% endraw %}
- 其中$W^{Q,K,V}_{i}$即图2右图的线性变换Linear部分，$shape=(512,64)$  
- $W^{o}$是对拼接后的向量做一次转换$shape=(64 * 8,512)=(512,512)$

### 讨论
多头注意力层是整个Transformer的核心。
- 注意力机制，是取代RNN循环结构的关键。解决了**并行计算**问题。
- 注意力机制，是解决序列建模问题的另一个有效手段：每个词都是其他词的加权平均，而且**没有距离衰减**（强于RNN）。这种每个词都蕴含其他词信息的建模方法非常契合自然语言这种抽象数据。
- 多头机制：和多卷积核类似，希望不同的头能够捕捉**不同的加权方式**。多头的核心是$W^{Q,K,V}_{i}$的不同，继而引起$Q_i,K_i,V_i$的不同。

## Layer归一化层
对多头Attention的输出+原输入X做一次层归一化  
{% raw %}
$$\begin{alignedat}{2}
output &= LayerNorm(X + MultiHead(X))\\
shape(X)&=(64,128,512)\\
shape(MultiHead(X))&=(64,128,512)
\end{alignedat}$$
{% endraw %}  
此处假设$batchsize=64$,每个样本最多128个token  
### LayerNorm VS BatchNorm
$LayerNorm$:对每一个样本做归一化，本例中即对每个token的embedding做归一化，结果是每个token的embedding均值=0，方差=1  
$BatchNorm$:对一个batch中每个特征做归一化，结果是这个batch中，每个特征均值=0，方差=1  

Norm是要做的，因为好处较多，例如缓解梯度消失,加速训练等。
但文本数据一个batch中，每个序列长度大概率是不同的。这样对BN就不很友好。所以LN在NLP中比较常见

### 讨论
此处关键在于使用了ResNet的残差机制，当Encoder，Decoder模块变多时，残差模块有助于缓解梯度消失/爆炸等问题

#### 后续改进
**1. 归一化公式改变**  
Transformer中的layerNorm公式计算如下：
$$y=\frac{x-Mean(x)}{\sqrt{Var(x)+\epsilon}}\ast W+B$$
在Llama中，改进为RMSNorm：
$$y=\frac{x}{\sqrt{Mean(x^2)+\epsilon}}\ast W$$
LayerNorm的中心偏移没什么用(减去均值等操作)。将其去掉之后，效果几乎不变，但是速度提升了40%  

**2. 残差结构改变**  
Transformer中是：$output = LayerNorm(X + MultiHead(X))$  

Llama中改为：$output = X + MultiHead(RMSNorm(X))$

## 前馈层
即2层MLP，使用ReLU激活  
{% raw %}
$$\begin{alignedat}{2}
FFN(X)&=ReLu(XW_{1}+b_1)W_{2}+b_2\\
shape(W_1)&=(512,2048)\\
shape(W_2)&=(2048,512)
\end{alignedat}$$
{% endraw %} 

### 讨论
前馈层结构很简单，一个2层的MLP，中间维度放大4倍。
但是这个结构不可或缺：
- 仔细看多头注意力，其实都是线性的，更注重的是token间的信息组合
- 前馈层引入了非线性，增强了网络表达能力，注重了每个token/embedding的特征组合
- 前馈网络维度扩大到4倍再返回1倍，使得模型可以把特征在较高维度学习之后再回归原始维度
- 前馈网络的输入是多头注意力的拼接，经过前馈之后，各头之间的信息可以相互融合平衡。

归纳一下，其作用主要是：**引入非线性，融合各头信息，提高表达能力（引入更高维度空间）**

## 输入层
$$X = Embedding + Positions Encoding$$

### 语义编码 Embedding
普通的可训练的embedding

### 位置编码 Positions Encoding
因为self-Attention是无所谓句子内token顺序的，即"hello, how are you" 和 "are how , you hello"的输出其实是一样的。  
因此，初始的embedding需要加入位置信息
位置信息如何加入到embedding里面？  
根本上这是一种对**数字作embedding**的问题  
{% raw %}
$$\begin{alignedat}{2}
PE_{(pos,2i)} &= sin(\frac{pos}{10000^{\frac{2i}{d_{model}}}})\\
PE_{(pos,2i+1)} &= cos(\frac{pos}{10000^{\frac{2i}{d_{model}}}})
\end{alignedat}$$
{% endraw %} 

针对句子中第1个token（pos=0）的embedding:
{% raw %}
$$\begin{alignedat}{2}
PE_{(0,0)} &= sin(\frac{0}{10000^{\frac{0}{512}}})=sin(0)=0\\
PE_{(0,1)} &= cos(\frac{0}{10000^{\frac{0}{512}}})=cos(0)=1\\
PE_{(0,2)} &= sin(\frac{0}{10000^{\frac{2}{512}}})=sin(0)=0\\
PE_{(0,3)} &= cos(\frac{0}{10000^{\frac{2}{512}}})=cos(0)=1\\
&...\\\
PE_{(0,510)} &= sin(\frac{0}{10000^{\frac{510}{512}}})=sin(0)=0\\
PE_{(0,511)} &= cos(\frac{0}{10000^{\frac{510}{512}}})=cos(0)=1\\
\end{alignedat}$$
{% endraw %} 
针对句子中第11个token（pos=10）的embedding:
{% raw %}
$$\begin{alignedat}{2}
PE_{(10,0)} &= sin(\frac{10}{10000^{\frac{0}{512}}})=sin(10)=0.17\\
PE_{(10,1)} &= cos(\frac{10}{10000^{\frac{0}{512}}})=cos(10)=0.98\\
PE_{(10,2)} &= sin(\frac{10}{10000^{\frac{2}{512}}})=sin(9.65)=0.16\\
PE_{(10,3)} &= cos(\frac{10}{10000^{\frac{2}{512}}})=cos(9.65)=0.98\\
&...\\\
PE_{(10,510)} &= sin(\frac{10}{10000^{\frac{510}{512}}})=sin(0.001)=0.0001\\
PE_{(10,511)} &= cos(\frac{10}{10000^{\frac{510}{512}}})=cos(0.001)=0.9999\\
\end{alignedat}$$
{% endraw %} 

正因为position encoding有很多0，1值，而token embedding大多是很小的数，
当两者相加时，为了不让position encoding占据过多信息，
就需要让token embedding整体扩大一些。故乘以$\sqrt{d_{model}}=\sqrt{512}$

# 讨论
Transformer提出时，是应用于机器翻译任务的，毕竟其本身也是Encoder-Decoder架构。
但是人们迅速意识到了其结构的优势，他是独立于CNN，RNN的第三种神经网络结构，相比RNN更能发挥硬件算力。

## 3种Attention结构
### 1. Encoder中的Self-Attention
完全可并行，**双向注意力模式**。  
该结构的注意力是双向的，即对于每一个词/token，计算时都会用到其前后词的信息。  
这种双向结构，使得其对每个**词的建模**更加精确，在应用上更侧重于**语言理解**，这也是Encoder的设计目的：充分理解输入。  
- 这也是BERT（Encoder-Only）更被广泛使用的原因-他能提供更好的词向量。

### 2. Decoder中的Masked-Self-Attention
**单向注意力模式**。  
在训练时，为了防止输出“偷看”到预测值，Decoder的输入Attention采用了Mask机制。  
即：当要预测$X_i$时，模型只能利用到$[X_0,X_1,...,X_{i-1}]$的信息。而$[X_i,X_{i+1},...,X_N]$的信息被Mask掉了。  
这使得该结构更专注于如何利用前面的信息。  
- GPT（Deocder-Only）就是只使用了这个结构，使其在**文本生成**方面更胜一筹

### 3. Decoder中的Cross-Attention
更接近Seq2Seq中的注意力机制，双向机制。
- $Q$即Decoder前一层的输出
- $K，V$即Encoder的输出  

这里就是Encoder信息被Decoder使用的具体实现。得益于残差机制，Decoder的Masked-Self-Attention输出可以直接向下传输。
在GPT中，因为没有Encoder，所以这部分注意力机制也没有。

