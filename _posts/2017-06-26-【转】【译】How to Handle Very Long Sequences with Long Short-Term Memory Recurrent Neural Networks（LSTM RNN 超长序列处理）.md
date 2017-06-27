---
layout: post
---

LSTM在处理长时间序列上表现很好。
如果像是时序预测和文本翻译一样每个输入都有一个输出，那么LSTM可以做的很好。但是当你有一个长的输入序列却只有一个或者一小段输出的时候，LSTM的性能就受到挑战。
这就是我们经常说的序列标注和序列分类。

by  Jason Brownlee

原文地址 http://machinelearningmastery.com/handle-long-sequences-long-short-term-memory-recurrent-neural-networks/



包括下面一些例子：

* 包含上千个词的文件情感分类（MLP）
* 包含上千个时间状态的脑电痕迹分类（Medicine）
* 上千条成对DNA序列的编码和非编码基因分类（bioinformatics）

这些所谓的分类任务在使用RNN的时候需要特殊处理。

在这篇文章里，你将能发现六种方式来处理超长序列分类问题。让我们开始吧。

## 1.Use Sequences As-Is（以序列的“现状”为准）

起始点是试用原始的未改变的序列信号，这样可能会导致超长训练时态问题。

更麻烦的是，尝试在超长输入序列上进行反向传播，会导致梯度消失，相应的，模型也就无法学习。

在实践中，如果使用大型LSTM模型的话，250-500个时间状态是一个比较常用也比较合理的区间。

## 2.Truncate Sequences（缩短序列）

处理超长时间序列一个常用的技术就是简单的缩短它们。

主要的做法就是有选择的移除输入序列开始或者结尾的时间状态。

这样可以让序列强制达到一个可处理的长度，当然代价就是损失了一部分数据。

这么做的风险就是，我们移除的数据很可能就是对预测有用的包含重要信息的数据。

## 3.Summarize Sequences（序列概述）

对于一些问题而言，对输入序列做概述是可行的。

比如说，假设输入序列是单词，那么我们就可以移除输入序列中的所有单词然后用指定的词频来代替。

可以构想的是，对它们在整个训练集中的排位频率（ranked frequency）保持观察，一定是高于某些固定值的（fixed value）。

对序列做概述可以让我们集中输入序列主要部分的问题上，而且可以显著的减少序列的长度。

## 4. Random Sampling（随机抽样）

用随机抽样的方式去概述序列是一个不太系统的方法。为了减少序列的长度，选择或者移除随机的时间状态。

Alternately，根据需要的长度，随机的相邻子序列将被选择出来组成一个新的采样序列，注意根据自己的需求来处理重叠或者不重叠的问题。

如果没有明显的系统的方法来减少序列长度的话，这种方法是比较合适的。

这种方法也经常用来增加数据，通过每一个输入序列来生成不同的输入序列。这样做可以在训练数据较少的时候很好的提高模型的鲁棒性。

## 5. Use Truncated Backpropagation Through Time（TBPTT）

与其基于整个序列来更新模型，不如用最后时间状态的子集来估算梯度。

这就是所谓的“缩短的基于时间的反向传播”，这种方法可以显著的加速长序列RNN的学习速度。

方法的原理是：允许提供的所有的序列作为输入并且计算前向传播，但是只用最后10个或者100个时间状态来估算梯度和更新权重。

一些现在的LSTM框架允许用户去指定用来更新的时间状态的步数，分离用于输入序列的时间状态。

For example:

- [The “truncate_gradient” argument in Theano](http://deeplearning.net/software/theano/library/scan.html).

## 6. Use an Encoder-Decoder Architecture（编码—解码）

可以用自动编码器来学习长序列，使其变为一个新的表述长度，然后根据想要的输出，用一个解码器网络来解释被编码的信号。

这可能包括一个用作预处理的无监督自动编码器，或者最近的用在自然语言翻译的[encoder-decoder LSTM style networks](http://machinelearningmastery.com/learn-add-numbers-seq2seq-recurrent-neural-networks/) 

再次强调，训练超长序列仍然可能非常困难，但是越复杂的结构应该会提供更多的影响和能力，特别是结合上面一个或者更多的技术的时候。

## Honorable Mentions and Crazy Ideas

该部分列举了一些还没有深入考虑的额外的想法。

* 将输入序列分割成固定长度的子集，然后用这些子集作为分离的特征来训练模型（平行输入序列）
* 建立双向LSTM，里面的每一个LSTM都使用一半的输入序列，然后最后将每一层的结果合并在一起。两层或多层可以合适的减少输入序列的长度。
* 使用 sequence-aware encoding schemes, projection methods, and even hashing ，目的是减少时间状态in less domain-specific ways.

## Further Reading

This section lists some resources for further reading on sequence classification problems:

- [Sequence labeling](https://en.wikipedia.org/wiki/Sequence_labeling) on Wikipedia.
- [A Brief Survey on Sequence Classification](http://dl.acm.org/citation.cfm?id=1882478), 2010

## Summary

在本文中，你接触到了当你训练RNN时怎么处理超长序列的方法。

具体一点，你学到了:

- 怎么使用缩短、概述和随机抽样的方法来减少序列的长度
- 怎么使用TBPTT来调整训练。
- 怎么使用编码解码结构来调整网络结构。