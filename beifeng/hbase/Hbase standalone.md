# Hbase standalone 

标签（空格分隔）： hbase

[toc]

---
#1）配置hbase
###1、修改hbase-env.sh
```
export JAVA_HOME=/opt/soft/modules/jdk1.7.0_67
export HBASE_MANAGES_ZK=false
```
###2、修改hbase-site.xml
```
	<property>
		<name>hbase.tmp.dir</name>
		<value>/opt/soft/cdh5/hbase-0.98.6-cdh5.3.6/data/tmp</value>
	</property>
	<property>
		<name>hbase.rootdir</name>
		<value>hdfs://cdh5.hadoop.com:8020/hbase</value>
	</property>
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>
	<property>
		<name>hbase.zookeeper.quorum</name>
		<value>cdh5.hadoop.com</value>
	</property> 
```

###3、启动
> bin/hbase-daemon.sh start master
bin/hbase-daemon.sh start regionserver

#2）Hbase基本语法
###1、help "create" 查看如何创建一张表

###2、create "user" , "info"  创建 一张表和一个列簇

###3、describe 'user' 查看一张表的结构


###4、在表中插入数据
```
list /*查看默认 数据库中的表*/



put 'user' , '1001' , 'info:name' , 'zhangsan' 
put 'user' , '1001' , 'info:age' , '18' 
put 'user' , '1001' , 'info:sex' , '1' 

put 'user' , '1002' , 'info:name' , 'lisi' 
put 'user' , '1002' , 'info:age' , '19' 
put 'user' , '1002' , 'info:sex' , '0' 
```
###5、查询
```
--扫描全表信息,针对小表
scan 'user'
--查询某个用户所有字段信息
get 'user' , '1001'
--查询某个用户单张表信息
get 'user' , '1001' , 'info:age'
范围查询 scan range 根据zookeeper
```

###6、删除数据(数据不会立即删除，当Hbase分块的时候才会真正删除)
```
delete 'user' , '1001' , 'info:age'
```

###6)Hbase保证数据不丢失
 client ---> hlog --> Store#memStore(内存) ---> storefile(缓存文件)
             （WAL）                      flash      
如果 没有flash，这台机器就挂掉了，这时候 hbase就会从hlog中读取数据，放入另一台机器内存中，当达到固定大小就会在进行flash

###7)Hbase网站登录查看
http://cdh5.hadoop.com:60010/master-status

###8)其他常用Hbase常用命令
```
list_namespace /*查看所有的namespace*/
create_namespace 'gary' /*创建一个namespace*/
list_namespace_tables 'hbase' /*查询hbase命名空间下的所有表*/

```