---
title: PySpark Cheet Sheet
author: 王哲峰
date: '2022-07-01'
slug: spark-pyspark-cheet-sheet
categories:
  - spark
tags:
  - tool
---

<style>
details {
    border: 1px solid #aaa;
    border-radius: 4px;
    padding: .5em .5em 0;
}
summary {
    font-weight: bold;
    margin: -.5em -.5em 0;
    padding: .5em;
}
details[open] {
    padding: .5em;
}
details[open] summary {
    border-bottom: 1px solid #aaa;
    margin-bottom: .5em;
}
</style>

<details><summary>目录</summary><p>

- [Spark](#spark)
- [Spark 初始化](#spark-初始化)
  - [SparkContext 创建](#sparkcontext-创建)
  - [SparkContext 配置](#sparkcontext-配置)
  - [SparkContext 查看](#sparkcontext-查看)
  - [Shell 初始化 Spark](#shell-初始化-spark)
- [加载数据](#加载数据)
  - [Parallelized Collections](#parallelized-collections)
  - [加载外部数据](#加载外部数据)
- [检索 RDD 信息](#检索-rdd-信息)
  - [基本信息](#基本信息)
  - [统计信息](#统计信息)
- [Applying 函数](#applying-函数)
- [数据筛选](#数据筛选)
  - [获取数据](#获取数据)
  - [随机采样](#随机采样)
  - [筛选数据](#筛选数据)
- [迭代](#迭代)
- [数据重塑](#数据重塑)
  - [Reducing](#reducing)
  - [Grouping by](#grouping-by)
  - [Aggregating](#aggregating)
- [数学操作](#数学操作)
- [排序](#排序)
- [Repartitioning](#repartitioning)
- [保存](#保存)
- [Stopping SparkContext](#stopping-sparkcontext)
- [Execution](#execution)
</p></details><p></p>


# Spark

PySpark is the Spark Python API that exposes the Spark programming model to Python.

# Spark 初始化

## SparkContext 创建

```python
from pyspark import SparkContext

sc = SparkContext(master = "local[2]")
```

## SparkContext 配置

```python
from pyspark import SparkConf, SparkContext

conf = (
   SparkConf()
      .setMaster("local")
      .setAppName("My app")
      .set("spark.executor.memory", "lg")
)
sc = SparkContext(conf = conf)
```

## SparkContext 查看

```python
from pyspark import SparkContext

sc = SparkContext(master = "local[2]")
sc.version                # Retrieve SparkContext version
sc.pythonVer              # Retrieve Python version
sc.master                 # Master URL to connect to
str(sc.sparkHome)         # Path where Spark is installed on worker nodes
str(sc.sparkUser())       # Retrieve name of the Spark User running SparkContext
sc.appName                # Return application name
sc.applicationId          # Retrieve application ID
sc.defaultParallelism     # Return default level of parallelism
sc.defaultMinPartitions   # Default minimum number of partitions for RDDs
```

## Shell 初始化 Spark

```bash
$ ./bin/sparkshell master local[2]
$ ./bin/pyspark master local[4] pyfiles copy.py
```

# 加载数据

## Parallelized Collections

```python
from pyspark import SparkContext

sc = SparkContext(master = "local[2]")

rdd = sc.parallelize([("a", 7), ("a", 2), ("b", 2)])
rdd2 = sc.parallelize([("a", 2), ("d", 1), ("b", 1)])
rdd3 = sc.parallelize(range(100))
rdd4 = sc.parallelize([("a", ["x", "y", "z"]), ("b", ["p", "r"])])
```

## 加载外部数据

```python
from pyspark import SparkContext

sc = SparkContext(master = "local[2]")

textFile = sc.textFile("/my/directory/*.txt")
textFile2 = sc.wholeTextFiles("/my/directory/")
```

# 检索 RDD 信息

## 基本信息

```python
rdd.getNumPartitions()
rdd.count()
rdd.countByKey()
rdd.countByValue()
rdd.collectAsMap()
rdd3.sum()
sc.parallelize([]).isEmpty()
```

## 统计信息

```python
rdd3.max()
rdd3.min()
rdd3.mean()
rdd3.stdev()
rdd3.variance()
rdd3.histogram(3)
rdd3.stats()
```

# Applying 函数

```python
rdd.map(lambda x: x + (x[1], x[0])).collect()
rdd5 = rdd.flatMap(lambda x: x + (x[1], x[0]))
rdd5.collect()
rdd4.flatMapValues(lambda x: x).collect()
```

# 数据筛选

## 获取数据

```python
rdd.collect()
rdd.take(2)
rdd.first()
rdd.top(2)
```

## 随机采样

```python
rdd3.sample(False, 0.15, 81).collect()
```

## 筛选数据

```python
rdd.filter(lambda x: "a" in x).collect()
rdd5.distinct().collect()
rdd.keys().collect()
```

# 迭代

```python
def g(x):
    print(x)

rdd.foreach(g)
```

# 数据重塑

## Reducing

```python
rdd.reduceByKey(lambda x, y: x + y).collect()
rdd.reduce(lambda a, b: a + b)
```

## Grouping by

```python
rdd3.groupBy(lambda x: x % 2).mapValues(list).collect()
rdd.groupByKey().mapValues(list).collet()
```

## Aggregating

```python
seqOp = (lambda x, y: (x[0] + y, x[1] + 1))
combOp = (lambda x, y: (x[0] + y[0], x[1] + y[1]))
rdd3.aggregate((0, 0), seqOp, combOp)
rdd.aggregateByKey((0, 0), seqOp, combOp).collect()
rdd3.fold(0, add)
rdd.foldByKey(0, add).collect()
rdd3.keyBy(lambda x: x + x).collect()
```

# 数学操作

```python
rdd.subtract(rdd2)
rdd2.subtractByKey(rdd)
rdd.cartesian(rdd2).collect()
```

# 排序

```python
rdd2.sortBy(lambda x: x[1]).collect()
rdd2.sortByKey().collect()
```

# Repartitioning

```python
rdd.repartition(4)
rdd.coalesce()
```

# 保存

```python
rdd.saveAsTextFile("rdd.txt")
rdd.saveAsHadoopFile("hdfs://namenodehost/parent/child", "org.apache.hadoop.mapred.TextOutputFormat")
```

# Stopping SparkContext

```python
sc.stop()
```

# Execution

```bash
$ ./bin/sparksubmit examples/src/main/python/pi.py
```
