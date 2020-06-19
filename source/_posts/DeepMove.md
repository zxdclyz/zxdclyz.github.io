---
title: 读论文：DeepMove
date: 2020-06-17 20:37:10
category:
  - 读论文
tags:
  - DeepMove
  - RNN
  - Attention
mathjax: true
---

原文：DeepMove: Predicting Human Mobility with Attentional Recurrent Network

<!-- more -->

> 本文在有关人类移动预测的 RNN 模型中首次引入了注意力机制，通过对历史数据的分析来获取有关周期性的信息

## 问题的提出与定义

### 研究问题

人类移动行为的预测在如今有着很广泛的应用，例如智能交通、通信资源管理，打车平台等商业领域也十分依赖它。而过去的研究表明，根据个人移动轨迹的熵的测量，93%的人类移动行为是可以预测的。

目前为止，关于这方面的研究主要有几个方向：

- pre-defined pattern：通过观察人类移动轨迹预先定义好固定的行为模式，然后根据该模式进行预测
- model-based：使用诸如马尔科夫链、RNN 的模型来捕捉规律

前者方法比较古老，而且受到其预定义的模式的单一性的限制，效果并不如后者。但是 model-based 方法目前也有着一些问题：

- 人类的移动行为具有**复杂的顺序性规律**，并且依赖于时间
- 人类的移动行为具有**多级周期性**，例如每天，每周，每年的规律，甚至个人习惯
- 获得的轨迹**数据是非均匀并且稀疏的**，采样率较低

### 问题的定义

- 轨迹序列（Trajectory Sequence）：首先定义时空点$q$为一个时间戳$t$和地点$l$的元组$q=(t,l)$，对于给定的用户标识$u$，轨迹序列为一个时空点的序列$S^u=q_1q_2\dots q_n$

- 轨迹（Trajectory）：对于给定的轨迹序列$S^u$和一个时间窗$t_w$，轨迹是一个子序列

  $$
  S^u_{t_w}=q_iq_{i+1}\dots q_{i+k}\quad for\,1<j\leq k,t_{q_j}\in t_w\quad
  $$

  即在时间窗中的子序列

- 移动预测问题（Mobility prediction）：根据目前的轨迹
  $$
  S^u_{t_{w_m}}=q_1\dots q_n
  $$
  和历史的轨迹
  $$
  S^u_{t_{w_1}},S^u_{t_{w_2}}\dots S^u_{t_{w_{m-1}}}
  $$
  来预测下一个时空点 $q_{n+1}$

## DeepMove Model

### 总体思想

针对前面提到的 model-based 方法的问题，DeepMove 做出了一些改进，其思想主要为：利用一个多模态的 emdedding 模块来将稀疏的地点、时间、用户信息编码成一个稠密的向量来表示，然后交由加入了 attention 机制的 RNN 来处理，其具体结构如图

<img src="http://121.36.88.131:12345/images/2020/06/17/DeepMove-arch.png" alt="DeepMove" style="zoom:67%;" />

### 结构与各模块实现

DeepMove 主要分为三个模块：Feature Extracting and Embedding、Recurrent Module and Historical Attention 以及 Prediction

- Feature Extracting and Embedding

  首先根据问题的定义我们得到的轨迹分为当前轨迹和历史轨迹两部分，在分别进行处理之前，这两者都先交由多模态的 embedding 模块进行处理，具体地，首先将各个信息数字化，然后转化为 one-hot 表示，再输入 embedding 模块处理为一个 dense vector

- Recurrent Module and Historical Attention

  - Recurrent Module

    RNN 部分的作用是来捕捉序列性信息和当前路径中的长期依赖关系，具体用 GRU 单元来实现，它接收 embedding 模块输出的当前轨迹的 dense vector 序列作为输入，然后将每一步的状态变量输出给 Historical Attention 和 Prediction 模块

  - Historical Attention Module

    这个模块中引入了注意力机制来捕捉人类移动行为的多级周期性，具体地，该模块选择与当前轨迹相关度最高的历史轨迹送入预测模块。它由两个部分组成，一个 attention candidate generator 来生成参与选择的成员，以及一个 attention selector 来比较各个参与选择的各个成员与当前轨迹的相似度，两个模块的实现如下：

    - Embedding Encode Module

      <img src="http://121.36.88.131:12345/images/2020/06/17/embed1.png" style="zoom:70%;"/>

      attention candidate generator 的一种实现方法，其原理为，先对一条轨迹进行 resize，使其时间维度长度固定，对应每一个时间的地点数据填充在相应的时间位置，即对每一个时间段，有一个地点集合，然后对地点集合进行采样（average，maximum，none），将得到的定长的向量输入到全连接层进而得到 attention candidate 向量

    - Sequential Encode Module

      <img src="http://121.36.88.131:12345/images/2020/06/17/embed2.png" />

      attention candidate generator 的另一种实现方法，同样地用 GRU 单元构成 RNN，将每一步输出作为 attention candidate，尽管他没有直接体现出周期性并且没有完全保留时空信息，但他可以提取出一些序列性的信息，并交由接下来的 attention selector 来进行周期性的捕捉。并且由于其提取方式与 Recurrent Module 相同，轨迹数据被投影到了相同的空间中，在接下来的比较中也比较有利

    - Attention Selector

      <img src="http://121.36.88.131:12345/images/2020/06/17/attention.png" style="zoom:80%;" />

      attention selector 通过计算 attention candidate 与此刻输入的 Recurrent Module 的相似度来生成一个 context vector 用来进行预测。其计算过程为

      $$
      f( {h_t,s})=tanh( {g_tW_s})\\
      \alpha_i=\sigma(f( {h_t,s_i}))\\
       {c_t}=\sum\alpha_i {s_i}
      $$

      其中$s$是历史轨迹生成的 attention candidate，$f$是相似度得分函数，$\sigma$是 softmax 函数，即我们用各个得分的 softmax 给 attention candidate 加权求和作为 context vector

- Prediction

  预测模块接收来自 Recurrent Module，Historical Attention Module 以及 Embedding Module 的输出作为输入，然后将其拼接，再输入到全连接网络，最终通过负采样的 softmax 输出，类似分类任务在预设的地点集合中选出预测的地点

### 进步

- 在该领域中首次引入了注意力机制，从而同时考虑了异构变化规律以及多级的周期性
- 设计了两种不同的注意力机制来与循环模块配合
- 效果优于 state-of-art 的模型至少 10%，并且表现出泛化性和鲁棒性，同时该方法十分直观，可以解释

## 模型效果

论文在三个具有代表性的数据集上进行了实验，结果如图，可以看出 DeepMove 的模型通过其对于历史轨迹的周期性提取得到了优于其他模型的效果

<img src="http://121.36.88.131:12345/images/2020/06/17/performance.png" style="zoom:80%;" />

此外，为了验证该结构确实起到了提取周期性的作用，文中还专门对于 historical attention 模块的输出进行了可视化来对比，如图（左上角为工作日和工作日对比，右下角为周末与周末对比），在两个数据集中，都可以看出左上角和右下角对角线上的颜色更深，即代表该模块认为当前工作日（周末）的轨迹与历史的工作日（周末）相关度比较高

<img src="http://121.36.88.131:12345/images/2020/06/17/period.png" style="zoom:67%;" />

## 疑问与感想

- 文中只展示了工作日、周末之类的每周的周期性，对于较为稀疏的每月、每年的周期性是否会有效果？（理论上会有，是否是被数据集限制了？）
- 这篇文章原文中讲解比较详细，并且清楚介绍了前置的基础知识，读起来比较容易理解
