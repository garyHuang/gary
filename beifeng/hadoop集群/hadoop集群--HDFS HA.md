# hadoop集群--HDFS HA

标签（空格分隔）：hadoop集群

[TOC]

---

##hdfs集群搭建，机器配置
 ![hdfs001.png-2.3kB][1]
namenode，关闭安全模式
bin/hadoop dfsadmin -safemode leave
删除本机损坏的块
bin/hadoop fsck -delete


## 配置hdfs-site.xml
```
    <!-- 配置hdfs的集群服务名称,本稳定中所用使用到ns都是这个集群的名称 -->
	<property>
	  <name>dfs.nameservices</name>
	  <value>ns</value>
	</property>
	<!-- 配置hdfs namenode的节点 -->
	<property>
	  <name>dfs.ha.namenodes.ns</name>
	  <value>nn1,nn2</value>
	</property>
	<!-- 配置hdfs 节点1 -->
	<property>
	  <name>dfs.namenode.rpc-address.ns.nn1</name>
	  <value>hadoop00.gary.com:8020</value>
	</property>
	<!-- 配置hdfs 节点2 -->
	<property>
	  <name>dfs.namenode.rpc-address.ns.nn2</name>
	  <value>hadoop01.gary.com:8020</value>
	</property>
	
	<!-- 配置hdfs namenode 通过http访问的端口 -->
	<property>
	  <name>dfs.namenode.http-address.ns.nn1</name>
	  <value>hadoop00.gary.com:50070</value>
	</property>
	<!-- 配置hdfs namenode 通过http访问的端口 -->
	<property>
	  <name>dfs.namenode.http-address.ns.nn2</name>
	  <value>hadoop01.gary.com:50070</value>
	</property>
	
	<!-- 配置hdfs 的datanode节点 -->
	<property>
	  <name>dfs.namenode.shared.edits.dir</name>
	  <value>qjournal://hadoop00.gary.com:8485;hadoop01.gary.com:8485;hadoop02.gary.com:8485/ns</value>
	</property>
	
	
	<property>
	  <name>dfs.client.failover.proxy.provider.ns</name>
	  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
	
	<property>
		<name>dfs.ha.fencing.methods</name>
		<value>sshfence</value>
	</property>
	
	<!-- 配置 hdfs启动密钥文件路径 -->
	<property>
	  <name>dfs.ha.fencing.ssh.private-key-files</name>
	  <value>/home/gary/.ssh/id_rsa</value>
	</property>
	<!--配置日志文件路径-->
	<property>
		<name>dfs.journalnode.edits.dir</name>
		<value>/opt/app/hadoop-2.5.0/data/dfs/jn</value>
	</property>

	<!-- 禁用权限控制 -->
	<property>
		<name>dfs.permissions.enabled</name>
		<value>false</value>
	</property>
	
	<!-- 配置自动故障转义 -->
	 <!-- ####start######## Automatic Failover ####### -->
	<property>
		<name>dfs.ha.automatic-failover.enabled.ns</name>
		<value>true</value>
	</property>
	<!-- ####end############ Automatic Failover ######## -->
```
## core-site.xml文件配置
```
    <!-- 默认的hdfs -->
<property>
	  <name>fs.defaultFS</name>
	  <value>hdfs://ns</value>
	</property>
	
	<!-- 配置文件保存的目录 -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/app/hadoop-2.5.0/data/tmp</value>
	</property>
	<!-- 配置默认访问的用户 -->
	<property>
		<name>hadoop.http.staticuser.user</name>
		<value>gary</value>
	</property>
	
	<!-- 将namenode 交给 zookeeper 管理  -->
	<!-- #########start############ NN HA Zookeeper ############################  -->
	<property>
		<name>ha.zookeeper.quorum</name>
		<value>hadoop00.gary.com:2181,hadoop01.gary.com:2181,hadoop02.gary.com:2181</value>
	</property>
	
```
## 配置slaves
```
hadoop00.gary.com
hadoop01.gary.com
hadoop02.gary.com
```
## 启动hdfs Ha
###1、使用命令 拷贝文件  scp -rp hadoop-2.5.0  hadoop01:/opt/app/
拷贝到其他两台机器上，
###2、然后启动每台机器的： JournalNode
    > sbin/hadoop-daemon.sh start journalnode
//这一步不能偷懒，否则将无法格式化namenode

###3、格式化namenode
```
// 格式化 hadoop00的namenode
bin/hdfs namenode -format
//然后启动 这台机的namenode
sbin/hadoop-daemon.sh start namenode
// 只在有namenode的起机器执行下列命令
bin/hdfs namenode -bootstrapStandby
//否则会报错信息如下：
```
![format-error.png-28.1kB][2]

###4、启动 DFSZKFailoverController
 这里要切记，有namenode的机器，就要启动 zkfc
 这里首先要先格式化集群zookeeper
  bin/hdfs zkfc -formatZK 
 > sbin/hadoop-daemon.sh start zkfc

### 5、最后启动 hdfs
> sbin/start-dfs.sh 


这是将可以看到 一个active的namenode和一个standby，当active的namenode出现问题的时候，zookeeper将自动启动 standby 的namenode

###6、使用jps查看进程：
 hadoop00
 ![hdfs jps 01.png-6.7kB][3]
 hadoop01
 ![hdfs jps 01.png-6.7kB][4]
 hadoop02
 ![hdfs jps 02.png-4.9kB][5]

###7、测试
```
 //上传word文件
 bin/hdfs dfs -put /opt/txt/wc.txt /opt/txt/
 
  bin/yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.0.jar wordcount data output
```


  [1]: http://static.zybuluo.com/Great-Chinese/td8wzain4g3v53n169qyxw9b/hdfs001.png
  [2]: http://static.zybuluo.com/Great-Chinese/x82sm3jpt220308yy31rt86k/format-error.png
  [3]: http://static.zybuluo.com/Great-Chinese/3er9qlneqag7d24e54dfq6ua/hdfs%20jps%2001.png
  [4]: http://static.zybuluo.com/Great-Chinese/3er9qlneqag7d24e54dfq6ua/hdfs%20jps%2001.png
  [5]: http://static.zybuluo.com/Great-Chinese/8xqi41hxuf42ayfe5cv03e44/hdfs%20jps%2002.png