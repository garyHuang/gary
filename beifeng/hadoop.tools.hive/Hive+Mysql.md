# Hive+Mysql

标签（空格分隔）： hadoop.tools.hive

[TOC]

---
*官网配置地址*
https://cwiki.apache.org/confluence/display/Hive/AdminManual+MetastoreAdmin
##1、安装mysql
```
   sudo yum install mysql*
```
####  安装启动mysqld
```
sudo  service mysqld start
```

####设置账号密码
```
update user set password = password('123456')  where user = 'root'; /*修改当前用户的密码为123456,一定要使用password函数，将密码进行加密，否则将无法登录mysql*/
 flush privileges;/*修改密码立即生效*/
```
##2、配置Hive
####配置创建hive-site.xml文件，并输入一下内容
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!--####连接Mysql##start###################-->
	<!-- 连接mysql的url
	
	参数说明 hadoop.gary.com 是主机名
	         metaStore 数据库名
	        createDatabaseIfNotExist 如果数据库不存在就创建
	-->
	<property>
	  <name>javax.jdo.option.ConnectionURL</name>
	  <value>jdbc:mysql://hadoop.gary.com/metaStore?createDatabaseIfNotExist=true</value>
	</property>
	
	<!-- 连接mysql的驱动-->
	<property>
	  <name>javax.jdo.option.ConnectionDriverName</name>
	  <value>com.mysql.jdbc.Driver</value>
	</property>

	<!--mysql 的账号-->
	<property>
	  <name>javax.jdo.option.ConnectionUserName</name>
	  <value>root</value>
	</property>
	<!-- mysql的密码 -->
	<property>
	  <name>javax.jdo.option.ConnectionPassword</name>
	  <value>123456</value>
	</property>
	<!--###########连接Mysql#end####################-->
</configuration>
```
####拷贝Mysql驱动Jar
将Mysql的驱动拷贝到 $HIVE_HOME\lib\ 目录下
![mysql-jar.png-80.5kB][1]

####启动Hive
配置后启动有点慢，因为他在连接mysql和创建默认数据库
```
 bin/hive 
```
当Hive启动成功后，可以登录mysql查看是否有配置的数据库“metaStore”，进入当前数据库查看相关表
![mysql-hive.png-12.7kB][2]
然后可以对Hive进行一些操作。
见文档： https://www.zybuluo.com/Great-Chinese/note/226288

  [1]: http://static.zybuluo.com/Great-Chinese/oki38eryppdop7pcf9bpskfw/mysql-jar.png
  [2]: http://static.zybuluo.com/Great-Chinese/xtosgx9atq0o0g948es109xy/mysql-hive.png