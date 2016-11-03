# spark

标签（空格分隔）： spark

[TOC]

---
Spark core
    >> 四大优势
     1.speed数组
     2.easy
     3.运行
        data location
        application running
    >>编译，可以根据hadoop和hive不同的版本去编译
    >>spark-shell 本地模式
    >>rdd 五大特性
       1.一个rdd有很多分区（a list of partitions）
       2.每一个分区进行计算(A function for computing each split)
       3.每一个rdd都会依赖其他的rdd(A list of dependencies on other RDDs)
       4.特殊rdd会进行key-value进行分区(Optionally, a Partitioner for key-value RDDs)
       5.每个分片找一个最佳的计算位置（Optionally, a list of preferred locations to compute each split on）
     >>spark调度
     dag->stage->Task->run on Exectuor
       
        
Spark弹性：
弹性之一：自动的进行内存和磁盘数据存储的切换
弹性之二：基于Lineage的高校容错
弹性之三：Task如果失败会自动进行特定次数的重试
弹性之四：Stage如果失败会自动进行特定次数的重试

使用缓存：特别耗时、计算链条、Shuffle之后、Checkpoint
![spark.png-251.5kB][1]

#一、编译Spark
##1) 检查网络 ping www.baidu.com
    关闭hadoop等相关服务
##2) 安装jdk和Maven
 jdk版本1.6+
 Maven版本3.0.4+
##3）Maven镜像（在setting.xml文件中添加maven仓库）
 如果编译不成功就添加

 
##4）google DNS
 /etc/resolv.conf 
 加入内容
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```
##5)解压spark，并修改make-distribution.sh文件第129行到131行
```
VERSION=1.3.0
SPARK_HADOOP_VERSION=2.5.0
SPARK_HIVE=1
SCALA_VERSION=2.10.4
```
##6）安装scala
A、解压scala
B、配置环境变量
输入scala -version提示如下，说明安装成功
![scala-env.png-2.8kB][2]
编译命令,使用此命令 需要删除setting.xml文件中设置的 maven仓库地址
> ./make-distribution.sh --tgz -Pyarn -Phadoop-2.4 -Dhadoop.version=2.5.0-cdh5.3.6 -Phive -Phive-thriftserver -Phive-0.13.1

看打这个界面 说明spark编译成功了，<span style="color:red">编译过程中scala会报很多警告，直接忽略就好。</span>
![spark-succ.png-46.5kB][3]    

#二、spark本地配置
##1)修改spark-env.sh，spark服务读取文件
```
JAVA_HOME=/opt/soft/modules/jdk1.7.0_67
SCALA_HOME=/opt/soft/cdh5/scala-2.10.4
HADOOP_CONF_DIR=/opt/soft/cdh5/hadoop-2.5.0-cdh5.3.6/etc/hadoop
SPARK_MASTER_IP=cdh5.hadoop.com
SPARK_MASTER_PORT=7077
SPARK_MASTER_WEBUI_PORT=8080
SPARK_WORKER_CORES=2
SPARK_WORKER_MEMORY=2g
SPARK_WORKER_PORT=8081
SPARK_WORKER_INSTANCES=1
SPARK_HISTORY_OPTS="-Dspark.history.fs.logDirectory=hdfs://cdh5.hadoop.com:8020/spark/logs"
##配置spark服务器端日志保存位置
```
##2修改spark-defaults.conf 客户端读取文件
```
spark.master					 spark://cdh5.hadoop.com:7077
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://cdh5.hadoop.com:8020/spark/logs
spark.serializer                 org.apache.spark.serializer.KryoSerializer
spark.driver.memory              4g
spark.eventLog.compress=true
```
启动spark服务
sbin/start-master.sh
sbin/start-slaves.sh
sbin/tart-history-server.sh
##2)启动spark本地模式
> bin/spark-shell

看到这个，就说spark本地模式启动成功
![spark-succ01.png-84.5kB][4]

##3）sc 对象基本API,这是需要启动Namenode和datanode
```
var rdd = sc.textFile("README.md") ;/*读取HDFS当前用户目录下的README.md文件*/
rdd.count /*读取文件的行数*/
rdd.count() /*读取文件行数Scala，调用无参数的方法 可以不带小括号*/
rdd.first /*读取文件的第一行内容*/

/*统计文件内包含的某个单词的数量的集中不同方式*/
rdd.filter((line: String) => line.contains("Spark")).count
rdd.filter((line) => line.contains("Spark")).count
rdd.filter(line => line.contains("Spark")).count
rdd.filter(_.contains("Spark")).count

/*scala wordcount*/
var array = sc.textFile("hdfs://cdh5.hadoop.com:8020/user/gary/README.md").flatMap(_.split(" ")).map((_,1)).reduceByKey((_+ _)).collect

/*查看debug读取信息*/
rdd.toDebugString
``` 


  [1]: http://static.zybuluo.com/Great-Chinese/t7bt9i1a92ybzfndguxz86xr/spark.png
  [2]: http://static.zybuluo.com/Great-Chinese/ehkjb2lwsaaonugrz7tzrsmd/scala-env.png
  [3]: http://static.zybuluo.com/Great-Chinese/gl4bfafbz2cybmqckxkter3o/spark-succ.png
  [4]: http://static.zybuluo.com/Great-Chinese/7k7p1b3wijvxj2lcydqe3fc0/spark-succ01.png