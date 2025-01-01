---
title: 源码解读-nlp-word2vec-gensim
mathjax: true
date: 2018-12-01 14:19:32
categories: 技术
tags:
    - word2vec
    - NLP
    - 源码解读
description: 训练词向量的方法有很多，当时使用python的gensim包，简单读了下源码
---
version=2.0.0
注意截止20181107，gensim最新版本已到了3.6 其源码与2.0差别很大，主要是结构上的区别，内容上差不多。

# 总体来说，训练过程执行两步
1，self.build_vocab
2，self.train

# build_vocab
其中，build_vocab函数里，依次执行三个函数 参考https://blog.csdn.net/u014568072/article/details/79071116,
## self.scan_vocab
`self.scan_vocab(sentences, progress_per=progress_per, trim_rule=trim_rule) #对句子中的单词进行初始化`
新建一个vocab字典
把所有句子（是一个个list，里面是一个个Word）扫描一遍，做WordCount。
其中，如果设置了self.max_vocab_size,那么，会依次删除vocab里词频最小的词
这个vocab最后赋值给 self.raw_vocab作为最原始的vocab，最最终，vocab要以self.wv.vocab为准

## self.scale_vocab
`self.scale_vocab(keep_raw_vocab=keep_raw_vocab, trim_rule=trim_rule, update=update) # 应用min_count的词汇表设置（丢弃不太频繁的单词）和sample（控制更频繁单词的采样）。`
有几个重要参数：
1. update：更新的是self.wv.vocab的值。即当前的raw_vocab用来更新self.wv.vocab
2. dry_run（一般默认false）：可以理解为是否要干跑，干跑即不更新self.wv.vocab等值，仅跑一下数据，统计一下信息，例如词频，估计内存等，会模拟着去除低频词，处理高频词，但不把处理的结果放进self.wv

**主体函数的作用**
1. 去低频词 min_count
2. 下采样高频词 sample，高频词在后续中每次的训练中，以一定的概率p被保留（即1-p的概率被舍弃）。这里涉及到self.wv.vocab['key'].sample_int这个值，这个值是Vocab类的一个属性，表示被保留的概率$p * 2 ** 32$，后续如果随机生成一个（0,$2 ** 32$）的数，如果这个数大于sample_int,则本次训练就舍弃本词。
3. 因为一般不会干跑，即会更新self.wv。此处，主要更新self.wv.vocab 和 self.wv.index2word
    - **self.wv.vocab**：是一个dict。键是词，值是一个Vocab类。类属性有count词频，index序号（对应self.wv.index2word里的值），sample_int被保留概率*2**32
    - **self.wv.index2word**：是一个list。里面存的是dict的键，即词。

## self.finalize_vocab
`self.finalize_vocab(update=update)  # build tables & arrays根据最终词汇表设置建立表格和模型权重。`
上一步中，已经把训练需要的vocab初始化并预处理完毕。本函数主要作用是为接下来的词向量训练做预备即：
1. 对于层次softmax方法(即`self.hs=1`)，生成哈夫曼树`self.create_binary_tree()`
2. 对于负采样方法(即`self.ns>0`)，生成负采样词表 `self.make_cum_table()`
3. 根据参数`self.sorted_vocab==1`对vocab排序，是按词频count排序，倒序排列。默认会排序，默认会倒序，这样在`mostsimilar`函数中参数`restrict_vocab`就排上了用场(该参数设置一个int，表示在求mostsimilar时，只使用前`restrict_vocab`个词做检索)
4. 初始化词向量（self.wv.syn0，对每个输入词，都哈希加随机一个初始向量），初始化权重（self.syn1--hs，self.syn1neg--ns）`self.wv.syn0norm = None`
       以及初始化一个`self.syn0_lockf = ones(len(self.wv.vocab), dtype=REAL)  # zeros suppress learning`

以一次训练日志来说一下：
``` log
2018-11-05 19:12:27,816: INFO: collecting all words and their counts  #scan_vocab方法。下面。。。是指实时处理信息
。。。
2018-11-05 19:12:28,921: INFO: collected 108425 word types from a corpus of 2130859 raw words and 82450 sentences  #scan_vocab方法。总结数据：108425个distinct词，语料包含2130859个词，82450个句子
2018-11-05 19:12:28,921: INFO: Loading a fresh vocabulary  #scale_vocab方法。开始
2018-11-05 19:12:29,187: INFO: min_count=3 retains 65034 unique words (59% of original 108425, drops 43391)#scale_vocab方法。去低频词。去掉词频<=3的词之后剩余65034个distinct词，即保留了59%的distinct词
2018-11-05 19:12:29,187: INFO: min_count=3 leaves 2061089 word corpus (96% of original 2130859, drops 69770)#scale_vocab方法。去低频词。 去掉词频<=3的词之后剩余2061089个词，保留了96%的语料词
2018-11-05 19:12:29,371: INFO: deleting the raw counts dictionary of 108425 items    #scale_vocab方法。删除row_vocab
2018-11-05 19:12:29,377: INFO: sample=0.001 downsamples 13 most-common words    #scale_vocab方法。下采样高频词，下采样参数是0.001，下采样了13个高频词
2018-11-05 19:12:29,377: INFO: downsampling leaves estimated 2007163 word corpus (97.4% of prior 2061089)    #scale_vocab方法。下采样高频词，下采样后预计语料变为2007163个词，保留了97.4%
2018-11-05 19:12:29,378: INFO: estimated required memory for 65034 words and 200 dimensions: 201605400 bytes    #scale_vocab方法。预计使用内存201605400 bytes  即200M
2018-11-05 19:12:29,451: INFO: constructing a huffman tree from 65034 words    #finalize_vocab方法。构建哈夫曼树。此处因为设置了hs=1
2018-11-05 19:12:31,718: INFO: built huffman tree with maximum node depth 19    #finalize_vocab方法。构建了最大深度为19的哈夫曼树
```
截止目前，vocab部分就搞定了。接下来是词向量训练`self.train`

# self.train
train这个函数内部包含或调用了很多子函数。大致分布如下
```python
train {
    1.检查参数是否合法
    2.调用多线程训练模型以及训练过程中的状态日志显示{
        2.1.work_loop{        #模型训练子函数，接收job_producer产生的job，从job获取sentences进行训练
            2.1.1.self._do_train_job{        #实际训练数据的函数
                            2.1.1.1.train_batch_sg{        #实现skip-gram模型。
                                    2.1.1.1.1.首先把输入的sentences逐句转换成word_vocabs顺便根据其sample_int值做了下采样。
                                    2.1.1.1.2.对于当前句的每一个词，它与其窗口内的每一个词(除了它自己)组成一对送入下面的函数
                                    2.1.1.1.3.train_sg_pair{        #对于上一步中的一对词(当前词w1->上下文词w2)做训练,这里w1是目标词，w2是输入词，有点反直觉
                                        if hs:则做hs
                                        if ns:则作ns  注意这里，hs和ns可以都做，默认ns=1，hs=0。如果只设置hs=1而没把ns设置成0的话，会都做。
                                    }
                            }
                            2.1.1.2.train_batch_cbow{        #实现cbow模型。
                                    2.1.1.2.1.首先把输入的sentences逐句转换成word_vocabs 顺便根据其sample_int值做了下采样。
                                    2.1.1.2.2.对于当前句的每一个词，组合(该词，上下文词之和)传入下函数
                                    2.1.1.2.3.train_cbow_pair{
                                        if hs:则做hs
                                        if ns:则作ns  注意这里，hs和ns可以都做，默认ns=1，hs=0。如果只设置hs=1而没把ns设置成0的话，会都做。
                                    }
                            }
                    }
        }
      2.2.job_producer{        
        产生一个job队列job_queue，
        持续从数据源读入sentences，
        然后sentences数到达指定值(default=1w)后，
        把这批sentences放入一个job，并把该job压入队列job_queue
      }
        2.3.使用2.1 2.2 组合多线程，由2.2产生一个个job(一个batch的sentences)，
            然后通过队列job_queue发送给2.1，由多个workloop训练模型。
            注意，此过程中，2.2只有一个，2.1有多个
    }
    3.后续检查
}
```
Python多线程与gensim多线程的问题：
1. Python多线程(multithreding)对于CPU密集型的计算没有提速效果。对于io密集型好一点。但整体不会好很多。如果要跑多任务，请使用多进程（multiprocess）
2. gensim为什么使用了多线程呢？这里由于内部是调用c++代码 所以还是能起到多线程作用

# 相似度计算：
1. 使用cos相似度
2. 词向量即syn0，而syn0norm即syn0归一化后的结果，归一化形式是每个词向量除以自己的模  
`self.syn0norm[i, :] = self.syn0[i, :]/sqrt((self.syn0[i, :] ** 2).sum(-1))`
3. 计算mostsimilar，直接使用syn0norm
4. 计算similar(x,y)，先使用syn0获取x,y词向量wx,wy，然后单独对wx,wy归一化，再计算。
