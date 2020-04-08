---
title: Learning Spark(1) 安装与基本概念
date: 2020-04-08 23:56:53
category:
  - Spark
tags:
  - Spark
  - data science
---

“Spark 是一个用来实现快速而通用的集群计算的平台”。

<!-- more -->

## Spark 的安装与使用

### 完整的 Spark 包

从官网上可以下载到 Spark 完整的预编译包，解压后即可使用，其中含有针对 python、R、Scala 的 shell，以及用来使用 spark 执行脚本的`spark-submit`

经过测试，在`pyspark`中默认 import 了`pyspark`包，相当于

```python
from pyspark import *
```

可以直接调用各种功能

### python 中的 pyspark

Spark 包中包含的 pyspark 是一个独立的 shell，而在 python 的包管理中也有 pyspark 这一个包，它与 pyspark shell 有什么区别？

- 安装了 pyspark 包之后也会附带 pyspark shell，并且设置了环境变量，可以直接启动
- 可以像其他包一样在 python 中 import 使用，但是调用时的输出表明它仍然是使用了 spark 的内核（有相同的输出）
- 在 Spark 包中包含的 pyspark 也可以直接 import 原本的环境中的模块

经以上实验，暂时可以得出结论，单独安装的 pyspark 包相当于是安装了只有 python 接口的 spark 内核，并且在使用到时猜测其用`spark-submit`的方式来运行脚本

## Spark 基本概念

### SparkContext(sc)

> 驱动器程序通过一个 SparkContext 对象来访问 Spark。这个对象代表对计算集群的一个连接

可以先简单的理解为创建了一个到集群服务器/本地的连接

### RDD

> Spark 对数据的核心抽象——弹性分布式数据集（Resilient Distributed Dataset，简称 RDD）

在下一节做具体介绍
