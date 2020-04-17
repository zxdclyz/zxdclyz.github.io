---
title: Learning Spark 3 键值对操作
date: 2020-04-17 19:31:50

category:
  - Spark
tags:
  - Spark
  - data science
---

键值对 RDD 是 Spark 中许多操作所需要的常见数据类型

<!-- more -->

## Pair RDD

Spark 为包含键值对类型的 RDD（Pair RDD） 提供了一些专有的操作

### 创建 Pair RDD

最简单的方法可以用 map 方法将普通 RDD 转换为 Pair RDD

```python
pairs = lines.map(lambda x: (x.split(" ")[0], x))
```

注意到这里返回的数据是一个（key,value）二元组

### Pair RDD 的转化操作

- Pair RDD 可以使用所有标准 RDD 上的可用的转化操作，但传递的函数应该考虑对二元组操作二元组，详见[文档](http://spark.apache.org/docs/latest/rdd-programming-guide.html#working-with-key-value-pairs)

  一个简单的例子为

  ```python
  result = pairs.filter(lambda keyValue: len(keyValue[1]) < 20)
  ```

- 可以通过`groupByKey()`函数来按 key 对数据进行分组，得到的结果为`[K, Iterable[V]]`的形式

- 常用的连接如`join(),leftOuterJoin(),rightOuterJoin()`，其概念与数据库中相同
- 可以使用`sortByKey()`等操作对数据排序

### Pair RDD 的行动操作

和转化操作一样，所有基础 RDD 支持的传统行动操作也都在 pair RDD 上可用，当然它还有一些额外的操作，详见[文档](http://spark.apache.org/docs/latest/rdd-programming-guide.html#working-with-key-value-pairs)

## 数据分区

这一部分的内容与数据的分布式存储以及操作的性能有关，暂时先略过，必要时再进行学习
