---
title: 大数据生态
author: 王哲峰
date: '2023-02-03'
slug: database-big-data
categories:
  - database
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
img {
    pointer-events: none;
}
</style>

<details><summary>目录</summary><p>

- [大数据技术生态](#大数据技术生态)
  - [Hadoop](#hadoop)
- [数据存储](#数据存储)
  - [HDFS](#hdfs)
- [中低速数据处理](#中低速数据处理)
  - [MapReduce](#mapreduce)
  - [Tez 和 Spark](#tez-和-spark)
  - [Pig 和 Hive](#pig-和-hive)
  - [Impala 和 Presto 以及 Drill](#impala-和-presto-以及-drill)
  - [Hive on Tez 和 SparkSQL](#hive-on-tez-和-sparksql)
- [高速数据处理](#高速数据处理)
  - [Storm](#storm)
  - [Flink](#flink)
  - [KV Store](#kv-store)
- [特特任务](#特特任务)
  - [Mahout 和 Protobuf ZooKeeper](#mahout-和-protobuf-zookeeper)
  - [Yarn](#yarn)
- [综述](#综述)
- [参考](#参考)
</p></details><p></p>


> 这段时间在知乎上看到一个关于大数据的技术生态的话题，感觉写的不错

# 大数据技术生态

## Hadoop

大数据本身是个很宽泛的概念，Hadoop 生态圈(或者泛生态圈)基本上都是为了处理超过单机尺度的数据处理而诞生的。

你可以把它比作一个厨房所以需要的各种工具。锅碗瓢盆，各有各的用处，互相之间又有重合。
你可以用汤锅直接当碗吃饭喝汤，你可以用小刀或者刨子去皮。但是每个工具有自己的特性，虽然奇怪的组合也能工作，
但是未必是最佳选择。

# 数据存储

大数据，首先你要能存的下大数据。传统的文件系统是单机的，不能横跨不同的机器。

## HDFS

HDFS(Hadoop Distributed File System)的设计本质上是为了大量的数据能横跨成百上千台机器，
但是你看到的是一个文件系统而不是很多文件系统。比如你说我要获取 `/hdfs/tmp/file1` 的数据，
你引用的是一个文件路径，但是实际的数据存放在很多不同的机器上。你作为用户，不需要知道这些，
就好比在单机上你不关心文件分散在什么磁道什么扇区一样。HDFS 为你管理这些数据。

# 中低速数据处理

存的下数据之后，你就开始考虑怎么处理数据。虽然 HDFS 可以为你整体管理不同机器上的数据，但是这些数据太大了。
一台机器读取成 T 上 P 的数据 ~~(很大的数据哦，比如整个东京热有史以来所有高清电影的大小甚至更大)~~，
一台机器慢慢跑也许需要好几天甚至好几周。

对于很多公司来说，单机处理是不可忍受的，比如微博要更新 24 小时热博，
它必须在 24 小时之内跑完这些处理。那么我如果要用很多台机器处理，我就面临了如何分配工作，
如果一台机器挂了如何重新启动相应的任务，机器之间如何互相通信交换数据以完成复杂的计算等等。
这就是 MapReduce/Tez/Spark 的功能。MapReduce 是第一代计算引擎，Tez 和 Spark 是第二代。

## MapReduce

MapReduce 的设计，采用了很简化的计算模型，只有 Map 和 Reduce 两个计算过程(中间用 Shuffle 串联)，
用这个模型，已经可以处理大数据领域很大一部分问题了。那什么是 Map 什么是 Reduce？

考虑如果你要统计一个巨大的文本文件存储在类似 HDFS 上，你想要知道这个文本里各个词的出现频率。
你启动了一个 MapReduce 程序。

* Map 阶段，几百台机器同时读取这个文件的各个部分，分别把各自读到的部分分别统计出词频，产生类似 `(hello, 12100次)`，
  `(world，15214次)` 等等这样的 `Pair`(我这里把 Map 和 Combine 放在一起说以便简化)；
* 这几百台机器各自都产生了如上的集合，然后又有几百台机器启动 Reduce 处理。
  Reducer 机器 A 将从 Mapper 机器收到所有以 A 开头的统计结果，
  机器 B 将收到 B 开头的词汇统计结果(当然实际上不会真的以字母开头做依据，
  而是用函数产生 Hash 值以避免数据串化。因为类似 X 开头的词肯定比其他要少得多，
  而你不希望数据处理各个机器的工作量相差悬殊)。然后这些 Reducer 将再次汇总，
  `(hello，12100)＋(hello，12311)＋(hello，345881)= (hello，370292)`。
  每个 Reducer 都如上处理，你就得到了整个文件的词频结果。
  
这看似是个很简单的模型，但很多算法都可以用这个模型描述了。
Map＋Reduce 的简单模型很黄很暴力，虽然好用，但是很笨重。

## Tez 和 Spark

第二代的 Tez 和 Spark 除了内存 Cache 之类的新 feature，本质上来说，
是让 Map/Reduce 模型更通用，让 Map 和 Reduce 之间的界限更模糊，数据交换更灵活，
更少的磁盘读写，以便更方便地描述复杂算法，取得更高的吞吐量。

## Pig 和 Hive

有了 MapReduce，Tez 和 Spark 之后，程序员发现，MapReduce 的程序写起来真麻烦。他们希望简化这个过程。
这就好比你有了汇编语言，虽然你几乎什么都能干了，但是你还是觉得繁琐。
你希望有个更高层更抽象的语言层来描述算法和数据处理流程。
于是就有了 Pig 和 Hive。

Pig 是接近脚本方式去描述 MapReduce，Hive 则用的是 SQL。它们把脚本和 SQL 语言翻译成 MapReduce 程序，
丢给计算引擎去计算，而你就从繁琐的 MapReduce 程序中解脱出来，用更简单更直观的语言去写程序了。

有了 Hive 之后，人们发现 SQL 对比 Java 有巨大的优势。一个是它太容易写了。
刚才词频的东西，用 SQL 描述就只有一两行，MapReduce 写起来大约要几十上百行。
而更重要的是，非计算机背景的用户终于感受到了爱：我也会写 SQL！
于是数据分析人员终于从乞求工程师帮忙的窘境解脱出来，工程师也从写奇怪的一次性的处理程序中解脱出来。
大家都开心了。Hive 逐渐成长成了大数据仓库的核心组件。甚至很多公司的流水线作业集完全是用 SQL 描述，
因为易写易改，一看就懂，容易维护。

## Impala 和 Presto 以及 Drill

自从数据分析人员开始用 Hive 分析数据之后，它们发现，Hive 在 MapReduce 上跑，~~真鸡巴慢！~~
流水线作业集也许没啥关系，比如 24 小时更新的推荐，反正 24 小时内跑完就算了。
但是数据分析，人们总是希望能跑更快一些。比如我希望看过去一个小时内多少人在充气娃娃页面驻足，
分别停留了多久，对于一个巨型网站海量数据下，这个处理过程也许要花几十分钟甚至很多小时。而这个分析也许只是你万里长征的第一步，
你还要看多少人浏览了跳蛋多少人看了拉赫曼尼诺夫的 CD，以便跟老板汇报，我们的用户是猥琐男闷骚女更多还是文艺青年／少女更多。
你无法忍受等待的折磨，只能跟帅帅的工程师蝈蝈说，快，快，再快一点！

于是 Impala，Presto，Drill 诞生了(当然还有无数非著名的交互 SQL 引擎，就不一一列举了)。
三个系统的核心理念是，MapReduce 引擎太慢，因为它太通用，太强壮，太保守，我们 SQL 需要更轻量，更激进地获取资源，
更专门地对 SQL 做优化，而且不需要那么多容错性保证(因为系统出错了大不了重新启动任务，如果整个处理时间更短的话，比如几分钟之内)。
这些系统让用户更快速地处理 SQL 任务，牺牲了通用性稳定性等特性。

## Hive on Tez 和 SparkSQL

如果说 MapReduce 是大砍刀，砍啥都不怕，那上面三个就是剔骨刀，灵巧锋利，但是不能搞太大太硬的东西。
这些系统，说实话，一直没有达到人们期望的流行度。因为这时候又两个异类被造出来了。
他们是 Hive on Tez / Spark 和 SparkSQL。它们的设计理念是，MapReduce 慢，
但是如果我用新一代通用计算引擎 Tez 或者 Spark 来跑 SQL，
那我就能跑的更快。而且用户不需要维护两套系统。这就好比如果你厨房小，人又懒，
对吃的精细程度要求有限，那你可以买个电饭煲，能蒸能煲能烧，省了好多厨具。

# 高速数据处理

上面的介绍，基本就是一个数据仓库的构架了。底层 HDFS，上面跑 MapReduce／Tez／Spark，
在上面跑 Hive，Pig。或者 HDFS 上直接跑 Impala，Drill，Presto。这解决了中低速数据处理的要求。
那如果我要更高速的处理呢？

如果我是一个类似微博的公司，我希望显示不是 24 小时热博，我想看一个不断变化的热播榜，更新延迟在一分钟之内，
上面的手段都将无法胜任。于是又一种计算模型被开发出来，这就是 Streaming(流)计算。

## Storm

Storm 是最流行的流计算平台。流计算的思路是，如果要达到更实时的更新，我何不在数据流进来的时候就处理了？
比如还是词频统计的例子，我的数据流是一个一个的词，我就让他们一边流过我就一边开始统计了。

流计算很牛逼，基本无延迟，但是它的短处是，不灵活，你想要统计的东西必须预先知道，
毕竟数据流过就没了，你没算的东西就无法补算了。因此它是个很好的东西，但是无法替代上面数据仓库和批处理系统。

## Flink

* TODO

## KV Store

还有一个有些独立的模块是 KV Store，比如 Cassandra，HBase，MongoDB 以及很多很多很多很多其他的(多到无法想象)。
所谓 KV Store 就是说，我有一堆键值，我能很快速滴获取与这个 Key 绑定的数据。比如我用身份证号，能取到你的身份数据。
这个动作用 MapReduce 也能完成，但是很可能要扫描整个数据集。而 KV Store 专用来处理这个操作，所有存和取都专门为此优化了。
从几个 P 的数据中查找一个身份证号，也许只要零点几秒。这让大数据公司的一些专门操作被大大优化了。
比如我网页上有个根据订单号查找订单内容的页面，而整个网站的订单数量无法单机数据库存储，我就会考虑用 KV Store 来存。

KV Store 的理念是，基本无法处理复杂的计算，大多没法 JOIN，也许没法聚合，没有强一致性保证(不同数据分布在不同机器上，
你每次读取也许会读到不同的结果，也无法处理类似银行转账那样的强一致性要求的操作)。但是丫就是快。极快。
每个不同的 KV Store 设计都有不同取舍，有些更快，有些容量更高，有些可以支持更复杂的操作。必有一款适合你。

# 特特任务

## Mahout 和 Protobuf ZooKeeper

除此之外，还有一些更特制的系统／组件，比如 

* Mahout 是分布式机器学习库
* Protobuf 是数据交换的编码和库
* ZooKeeper 是高一致性的分布存取协同系统
* 等等

## Yarn

有了这么多乱七八糟的工具，都在同一个集群上运转，大家需要互相尊重有序工作。
所以另外一个重要组件是，调度系统。

现在最流行的是 Yarn。你可以把他看作中央管理，好比你妈在厨房监工，
哎，你妹妹切菜切完了，你可以把刀拿去杀鸡了。只要大家都服从你妈分配，
那大家都能愉快滴烧菜。

# 综述

你可以认为，大数据生态圈就是一个厨房工具生态圈。为了做不同的菜，中国菜，日本菜，法国菜，你需要各种不同的工具。
而且客人的需求正在复杂化，你的厨具不断被发明，也没有一个万用的厨具可以处理所有情况，因此它会变的越来越复杂

# 参考

* [如何用形象的比喻描述大数据的技术生态？](https://www.zhihu.com/question/27974418/answer/38965760)