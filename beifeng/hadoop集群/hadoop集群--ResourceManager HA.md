# hadoop集群--ResourceManager HA

标签（空格分隔）： hadoop集群

[TOC]

---

本文配置在HDFSHA之上进行配置
https://www.zybuluo.com/Great-Chinese/note/222030

##配置yarn-site.xml
```
    <!--启用ResourceManager HA-->
    <property>
		<name>yarn.resourcemanager.ha.enabled</name>
		<value>true</value>
	</property>
	<!--设置RM集群的名称-->
	<property>
		<name>yarn.resourcemanager.cluster-id</name>
		<value>yarn-cluster</value>
	</property>
	<!--设置RM的子节点-->
	<property>
		<name>yarn.resourcemanager.ha.rm-ids</name>
		<value>rm1,rm2</value>
	</property>
	<!--设置RM1子节点-->
	<property>
		<name>yarn.resourcemanager.hostname.rm1</name>
		<value>hadoop01.gary.com</value>
	</property>
	<!--设置RM2子节点-->
	<property>
		<name>yarn.resourcemanager.hostname.rm2</name>
		<value>hadoop02.gary.com</value>
	</property>
	<!--配置zookeeper地址-->
	<property>
		<name>yarn.resourcemanager.zk-address</name>
		<value>hadoop00.gary.com:2181,hadoop01.gary.com:2181,hadoop02.gary.com:2181</value>
	</property>
	<!-- 配置RM自动恢复-->
	<property>
		<name>yarn.resourcemanager.recovery.enabled</name>
		<value>true</value>
	</property>
	<!-- RM store 类-->
	<property>
		<name>yarn.resourcemanager.store.class</name>
		<value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
	</property>
```
##启动Yarn
//启动zkfc服务
> sbin/hadoop-daemons.sh start zkfc

//启动nodemanager
> sbin/hadoop-daemons.sh start nodemanager

//启动resourcemanager
> sbin/hadoop-daemons.sh start resourcemanager

##测试
 杀死hadoop01的resourcemanager，看看是否会自动切换到hadoop02机器上