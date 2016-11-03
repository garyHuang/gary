# 11月10日 hadoop2.x理解

标签（空格分隔）： hadoop

---

[TOC]


## 1、yarn 理解

## 2、HDFS理解
   Hadoop Distributed File System (Hadoop分布式文件系统)
## 3、MapReduce理解
   Mapreduce分布式处理大数据，执行过程如下：
      input     -->  Map    -->     shuffle  ---> reduce ---> out
    输入数据  将输入数据转          数据打乱    合并数据   输出数据
              换成Map对象
 优点：
   1、 这个模型非常方便使用，即使是对于完全没有分布式程序的程序员也是如此。它隐藏了并行计算的细节，本地优化以及负载均衡。MapReduce运行开发人员使用自己熟悉的语言进行开发。如 java等
  2、对于大型的计算需求使用MapReduce可以非常轻松的完成
  3、通过MapReduce，应用程序可以在超过1000个节点的大型集群上运行，并且提供经过优化的错误容灾
 缺点：1、MapReduce运行时会产生大量的临时文件。
       2、不适合一般Web应用。




