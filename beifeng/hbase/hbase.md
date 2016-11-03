# hbase 

标签（空格分隔）： hbase

[TOC]

---
##配置hbase-site.xml
```
<configuration>
	<property>
		<name>hbase.rootdir</name>
		<value>hdfs://hadoop00.gary.com:8020/hbase</value>
	</property>

	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>
	<property>
		<name>hbase.zookeeper.quorum</name>
		<value>hadoop00.gary.com,hadoop01.gary.com,hadoop02.gary.com</value>
	</property>
</configuration>

```
##配置regionservers，配置子节点
```
hadoop01.gary.com
hadoop02.gary.com
```
##启动Hbase
bin/hbase shell
```
bin/start-hbase.sh 
```
这里HMaster,需要先启动hadoop集群和zookeeper集群
jps主节点可以看到 HMaster进程,从节点可以看到 Hregionservers这个进程。
##操作数据库相关代码
```
bin/hbase shell 
可输入hbase相关命令进行数据库操作
``
