---
title: Learning Spark(2) RDD基础
date: 2020-04-10 23:41:33
category:
  - Spark
tags:
  - Spark
  - data science
---

“RDD（Resilient Distributed Dataset） 是分布式的元素集合”

<!-- more -->

## RDD 基本操作

### 创建

可以从内存或者文件创建 RDD

- 用`SparkContext.parallelize()`从内存创建，详见[文档](http://spark.apache.org/docs/latest/rdd-programming-guide.html#parallelized-collections)
- 使用诸如`SparkContext.textFile()`方法从外部文件读取，详见[文档](http://spark.apache.org/docs/latest/rdd-programming-guide.html#external-datasets)，以及[API doc](http://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.SparkContext)

### 转化与行动

转化操作从已有的 RDD 中生成新的 RDD，行动操作在 RDD 进行计算并返回结果

详见[文档](http://spark.apache.org/docs/latest/rdd-programming-guide.html#rdd-operations)及[API doc](http://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.RDD)

## RDD 的重要性质

### 惰性计算

Spark 对于对 RDD 的转化操作采用惰性计算的机制，即 Spark 会将你对 RDD 进行的转化操作记录下来，但并不立刻计算，只有当某个 RDD 被调用了行动操作时，Spark 才会按照记录的谱系图计算出这个 RDD，然后进行行动。值得注意的是，在每次进行行动时，Spark 都会重新计算一个 RDD

### 持久化(缓存)

由于惰性计算的机制，在每次进行行动时，Spark 都会沿着谱系图重新计算一个 RDD，这在迭代去情况下会产生极大的开销，因此，我们可以通过缓存的方式来对 RDD 进行持久化，使用`RDD.persist()`来进行缓存
