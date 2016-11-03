# Flume 日志实时监控

标签（空格分隔）： flume

[TOC]

---
#1、flume介绍
&nbsp;&nbsp;&nbsp;&nbsp;Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。
#2、下载安装
##2.1下载地址
  http://archive.cloudera.com/cdh5/cdh/5/ 下载 apache-flume-1.5.0-cdh5.3.6-bin.tar.gz版本
  官网文档 
  http://archive.cloudera.com/cdh5/cdh/5/flume-ng/
##2.2 安装
  a.解压 apache-flume-1.5.0-cdh5.3.6-bin
  b.修改配置文件conf/flume-env.sh 设置JAVA_HOME
  ![install.flume.png-60.3kB][1]

#3、测试flume框架
#3.1创建文件test.conf
```
## define agent
a1.sources = r1
a1.channels = c1
a1.sinks = k1

## define sources
a1.sources.r1.channels = c1
a1.sources.r1.type = netcat
a1.sources.r1.bind = hadoop00.gary.com
a1.sources.r1.port = 4444

## define channels
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

## define sinks
a1.sinks.k1.channel = c1
a1.sinks.k1.type = logger
```
使用运行命令：
```
bin/flume-ng agent --conf conf --name a1 --conf-file conf/test.conf -Dflume.root.logger=INFO,console
```
启动后采用telnet测试flume是否安装成功,在telnet输入一些内容，在flume地方可以看到对应内容
```
#默认端口 4444
telnet hadoop00 4444
```
![flume.init.test.png-13.1kB][2]
![flume.init.test2.png-44.8kB][3]

#4、flume实时监控日志上传到hdfs上
创建文件flume-up-hdfs.conf
```

## define agent
a2.sources = r2
a2.channels = c2
a2.sinks = k2

## define sources
a2.sources.r2.channels = c2
a2.sources.r2.type = exec
a2.sources.r2.command = tail -f /opt/soft/app/hive-0.13.1-cdh5.3.6/logs/hive.log
a2.sources.r2.port = 4444
a2.sources.r2.shell = /bin/bash -c


## define channels
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

## define sinks
a2.sinks.k2.channel = c2

a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop01.gary.com:8020/user/flume/logs
a2.sinks.k2.hdfs.fileType=DataStream
a2.sinks.k2.hdfs.filePrefix=gary
#文件格式超文本
a2.sinks.k2.hdfs.writeFormat=Text
a2.sinks.k2.hdfs.batchSize=1000
#文件回滚大小为128000000个字节
a2.sinks.k2.hdfs.rollSize=128000000
#没小时自动回滚依次
a2.sinks.k2.hdfs.rollInterval=3600
#设置取消自动回滚
a2.sinks.k2.hdfs.rollCount=0
```
运行命令：
> bin/flume-ng agent --conf conf --name a2 --conf-file conf/flume-up-hdfs.conf -Dflume.root.logger=INFO,console

看到错误提示：
![error01.png-63.1kB][4]
找不到类：org/apache/hadoop/io/SequenceFile$CompressionType，说明hadoop相关包没有导入，需要导入hadoop相关包，导入下列四个jar包，可以从hadoop中找到
![flume-need-hadoop-hdfs-jars.png-6kB][5]
再次运行：
> bin/flume-ng agent --conf conf --name a2 --conf-file conf/flume-up-hdfs.conf -Dflume.root.logger=INFO,console

  [1]: http://static.zybuluo.com/Great-Chinese/tk8krs7j0n5m31r0ufp8rmji/install.flume.png
  [2]: http://static.zybuluo.com/Great-Chinese/z7lzhpsv4oqsfir0010x5snf/flume.init.test.png
  [3]: http://static.zybuluo.com/Great-Chinese/9v6ukwpuy0qsm72btjdxjvs5/flume.init.test2.png
  [4]: http://static.zybuluo.com/Great-Chinese/8rkslterly7v166gz7w1e5g9/error01.png
  [5]: http://static.zybuluo.com/Great-Chinese/l0ngkxem24mf71xeb61nsx90/flume-need-hadoop-hdfs-jars.png