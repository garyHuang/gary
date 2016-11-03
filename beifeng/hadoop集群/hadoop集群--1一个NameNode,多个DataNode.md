# hadoop集群--1一个NameNode,多个DataNode 

标签（空格分隔）： hadoop集群


[TOC]

---
##配置要求:
```
  1个resourcemanger
  3个nodemanager
  1个NameNode
  3个DataNode
```
##具体配置

###配置core-site.xml
```
<configuration>
	<!-- 配置默认的文件系统 -->
	 <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop00.gary.com:8020</value>
    </property>
    <!--本地目录-->
    <property>
      <name>hadoop.tmp.dir</name>
      <value>/tmp/hadoop-${user.name}</value>
    </property>
    <!--默认用户-->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>gary</value>
    </property>
</configuration>

```
###配置hdfs-site.xml
```
<configuration>
    <!--权限检查-->
    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>
    <!--副本数-->
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <!--secondary服务-->
    <property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>hadoop02.zc.com:50090</value>
    </property>
</configuration>
```
###配置slaves
```
hadoop.zc.com
hadoop01.zc.com
hadoop02.zc.com

```
###配置 yarn-site.xml
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!--配置ResourceManager-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop01.zc.com</value>
    </property>
    <!--启用日志聚集-->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <!--aggregation（日志聚集）保留时间，秒。-->
    <property>
        <name>yyarn.log-aggregation.retain-seconds</name>
        <value>100800</value>
    </property>
</configuration>
```
###配置mapred-site.xml
```
<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <!--历史日志服务内部地址,mapred-site.xml-->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop00.gary.com:10020</value>
    </property>
    <!--历史日志服务地址web,mapred-site.xml-->
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop00.gary.com:19888</value>
 </property>
```
## 复制至各个节点
 $ scp -r hadoop-2.5.0/ hadoop01:/opt/app/
## 启动集群
### 1）格式化HDFS

$ bin/hdfs namenode -format

### 2）启动namenode

>  $ sbin/hadoop-daemon.sh start namenode
###3）启动datanode（各个配置是datanode的节点都要启动）

>  $ sbin/hadoop-daemon.sh start datanode
 
### 4）启动resourcemanager

> $ sbin/yarn-daemon.sh start resourcemanager

### 5）启动nodemanager（各个nodemanager点都要启动）

> $ sbin/yarn-daemon.sh start nodemanager

## 测试集群

1）上传文件

$ bin/hdfs dfs -put /opt/txt/wc.input /opt/txt/
2）运行mapreduce
> $ bin/yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.0.jar wordcount /opt/txt/wc.input /opt/out/output1

3)查看结果
> bin/hdfs dfs -text /opt/out/output1/p*
![hdfs-test-result.png-10.5kB][2]

4)可以尝试查掉当前 active的namenode，观察standby的namenode是否回自动启动


  [1]: http://static.zybuluo.com/Great-Chinese/km3tzpog2fwjg9u887n348oe/hdfs%20jps%2002.png
  [2]: http://static.zybuluo.com/Great-Chinese/t79c8k2c1lwpd2qugkqm83zb/yarn-test-result.png