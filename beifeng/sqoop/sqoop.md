# sqoop

标签（空格分隔）： sqoop
    
[TOC]

---

# 1）架构概述
  a. 传统数据库和hadoop HDFS之间数据转移桥梁。如（HDFS，Hive，Hbase）数据转移。
  b. 利用Map task加快数据传输速度，批处理方式数据传输,只有Map任务，不会有Reduce任务
# 2）使用注意
    a.RDBMS端：
        1.jdbcurl，数据库连接
        2.user，连接用户
        3.password，连接密码
        4.table 使用表
    b.操作： 
        import：导入到hdfs上 可以是hive，hbase等
        export：导出到关系型数据库
    c.hadoop： 
        对HDFS而言:path，路径
        对hive而言：tabelname，表
        对hbase而言也是表
        
#3)下载地址
http://archive.cloudera.com/cdh5/cdh/5/
http://archive.cloudera.com/cdh5/cdh/5/sqoop-1.4.5-cdh5.3.6.tar.gz
下载sqoop-1.4.5-cdh5.3.6.tar.gz

#4)基本配置
    a.配置文件conf/sqoop-env.sh
```
export HADOOP_COMMON_HOME=/opt/soft/app/hadoop-2.5.0-cdh5.3.6

#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/opt/soft/app/hadoop-2.5.0-cdh5.3.6
#set the path to where bin/hbase is available
#export HBASE_HOME=
#Set the path to where bin/hive is available
export HIVE_HOME=/opt/soft/app/hive-0.13.1-cdh5.3.6

export ZOOKEEPER_HOME=/opt/soft/app/zookeeper-3.4.5-cdh5.3.6
```
   b.拷贝mysqljar包到lib目录下
   c.测试连接mysql数据库
```
bin/sqoop list-databases --connect jdbc:mysql://hadoop00.gary.com:3306 --username root --password 123456
```
成功提示：
![sqoop001.png-22.8kB][1]
#5）测试导入数据到hive
[参考文档][2]： http://archive.cloudera.com/cdh/3/sqoop/SqoopUserGuide.html#_literal_sqoop_import_literal 

#5)测试导入数据
直接导入到导入hdfs
```
bin/sqoop import \
--connect jdbc:mysql://hadoop00.gary.com:3306/test \
--username root \
--password 123456 \
--table partgroup \
--split-by id \
--num-mappers 1 \
--target-dir /user/hive/warehouse/hive_gary.db/partgroup \
--fields-terminated-by "\t" \
--delete-target-dir 
```
导入到hive
```
###import hive
bin/sqoop import \
--connect jdbc:mysql://hadoop.gary.com:3306/test \
--username root \
--password 123456 \
--table partgroup \
--num-mappers 1 \
--fields-terminated-by "\t" \
--delete-target-dir \
--hive-database hive_gary \
--hive-import \
--hive-table partgroup
```
用查询语句导入到HDFS
```
bin/sqoop import \
--connect jdbc:mysql://hadoop.zc.com:3306/test \
--username root \
--password 123456 \
--query 'select * from my_user WHERE $CONDITIONS' \
--num-mappers 1 \
--target-dir /user/hive/warehouse/hive_gary.db/partgroup \
--fields-terminated-by "\t" \
--delete-target-dir 
```
#6）导出
导出HDFS
```
bin/sqoop export \
--connect jdbc:mysql://hadoop.gary.com:3306/test \
--username root \
--password 123456 \
--table partgroup \
--num-mappers 1 \
--input-fields-terminated-by "\t" \
--export-dir /user/hive/warehouse/hive_gary.db/partgroup
```
导出hdfs
```
bin/sqoop export \
--connect jdbc:mysql://hadoop.gary.com/test \
--username root \
--password 123456 \
--table user_export \
--num-mappers 1 \
--input-fields-terminated-by "\t" \
--export-dir /user/hive/warehouse/hive_gary.db/partgroup
```

#7)sqoop导入Mysql到Hbase
参考文档：http://sqoop.apache.org/docs/1.4.5/SqoopUserGuide.html#_importing_data_into_hbase
![remark.png-53kB][3]
##7.1修改配置文件$SQOOP_HOME/conf/sqoop-env.sh
> export HBASE_HOME=/opt/soft/cdh5/hbase-0.98.6-cdh5.3.6

执行命令
首先在Hbase中创建列簇： create 'parts' , 'info'
> bin/sqoop import --connect jdbc:mysql://***:3306/hks --username test --password test --table a_area_price --hbase-table table --column-family info --hbase-row-key id

  [1]: http://static.zybuluo.com/Great-Chinese/tiiybgsx792atjjzoonnr74k/sqoop001.png
  [2]: http://archive.cloudera.com/cdh/3/sqoop/SqoopUserGuide.html#_literal_sqoop_import_literal
  [3]: http://static.zybuluo.com/Great-Chinese/ix886haf221zalf75cke30dv/remark.png