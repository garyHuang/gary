#Hive创建表的不同方式+beeline

标签（空格分隔）： hadoop.tools.hive

[toc]

---
##准备数据
1	张三	18	技术部门	技术主管	20000
2	李四	20	技术部门	业务人员	15000
3	王五	25	技术部门	美工	8000
4	Gary	24	技术部门	业务人员	12000
5	Amy	35	销售部门	销售经理	25000
6	Rom	53	销售部门	销售人员	18000 
以UTF-8的编码保存后，上传到linux，否是会有乱码
## bin/hive进入hive环境
> bin/hive

![hive113001.png-4kB][1]
> show databases ;

![hive113002.png-4kB][2]
我这里已经创建了一个数据库,创建数据库语法
> create database if not exists hive_1130  ; 

使用新创建的数据库
> use hive_1130;

##创建表的第一种方式
```
CREATE TABLE IF NOT EXISTS emp(
id string COMMENT 'table id ',
name string COMMENT 'nae',
age string comment 'age',
dep string ,
job string,
sal string
)
COMMENT 'emp table row'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS textfile ;
```
## 创建表第二种方式
```
CREATE TABLE IF NOT EXISTS emp_copy like emp ;
```
##创建表第三种方式
```
CREATE TABLE IF NOT EXISTS emp_copy2 as select id,name from emp; 
```
##分开查看三张表的结构
![hive113003.png-15kB][3]

##创建外部表
```
CREATE EXTERNAL TABLE IF NOT EXISTS emp_ext(
id string COMMENT 'table id ',
name string COMMENT 'nae',
age string comment 'age',
dep string ,
job string,
sal string
)
COMMENT 'emp table row'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS textfile  tblproperties ("orc.compress"="SNAPPY")
LOCATION '/user/gary/hive/warehouse/emp/' ;
// LOCATION 指定表，在hdfs的位置
```
 

![hive_external.png-18.6kB][4]
现在已经创建的内部表：emp（数据文件在hdfs上的位置：/user/hive/warehouse/emp/），外部表：emp_ext(数据文件在hdfs上的位置：/user/gary/hive/warehouse/emp/)。两张表的区别是，删除内部表的时候会删除表在hdfs上的数据，删除外部表的时候不会删除表在hdfs的数据，现在测试删除内部表
> drop table if exists emp

登录hdfs上查看
![drop table if exists emp][5]
删除外部表
>  drop table if exists emp_ext ; 

登录hdfs上查看
![hive_drop_table_02.png-20.1kB][6]

##Hive创建表分区分区
###创建语法，用访问tomcat的url作为测试实例
用月份和日期进行分区
```
create EXTERNAL table if not exists tomcat_log(
 ip string , 
 c string,
 c1 string,
 date string,
 url string,
 code string,
 size string
)
partitioned by(month string,day string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS textfile 
LOCATION '/user/gary/hive/warehouse/tomcat_log/' ;
```
用hdfs上传一个日志文件到 /user/gary/hive/warehouse/tomcat_log/mouth=201511/day=14
这个目录。
> 
$ bin/hdfs dfs -mkdir -p /user/gary/hive/warehouse/tomcat_log/month=201511/day=14
$ bin/hdfs dfs -mkdir -put /opt/txt/logs/localhost_access_log.2015-11-14.txt /user/gary/hive/warehouse/tomcat_log/month=201511/day=14

依次多上传几个日志文件到hdfs对应的目录下


###然后在hive下输入命令：
> alter table tomcat_log add partition(month='201511',day='14') ; 

这样Hive表分区就完成了

## 启动hiveServer2服务，采用beeline访问hive
启动hiveserver2
> bin/hiveserver2

![hiveserver2.png-3kB][7]

bin/beeline命令，进入beeline命令行
![beeline.png-5.1kB][8]
在bin/beeline命令行输入下列命令，连接hiveserver2
> !connect jdbc:hive2://localhost:10000 scott tiger org.apache.hive.jdbc.HiveDriver


##创建表语句详细说明：
```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
  [(col_name data_type [COMMENT col_comment], ...)]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
  ]
  [LOCATION hdfs_path]
  [AS select_statement];  
  说明：
  - EXTERNAL：外部表
  - IF NOT EXISTS：表不存在创建
  - db_name：表所属数据库
  - COMMENT col_comment：列注释
  - COMMENT table_comment：表注释
  - PARTITIONED BY：分区字段
  - ROW FORMAT row_format:行的数据格式
  - STORED AS file_format:文件存储格式
  - STORED AS file_format
  - LOCATION hdfs_path：存放路径
  - AS select_statement：查询语句为结果集

CREATE TABLE [IF NOT EXISTS] [db_name.]table_name
  LIKE existing_table_or_view_name
  [LOCATION hdfs_path];
  说明：
  - IF NOT EXISTS：表不存在创建
  - db_name：表所属数据库
  - existing_table_or_view_name：结果集为存在的表或者师徒
  - LOCATION hdfs_path：存放路径
```

![beeline2.png-8.6kB][9]
连接hiveserver2默认端口为10000,可以设置hiveserver2的启动端口

  [1]: http://static.zybuluo.com/Great-Chinese/b9bjbldh160a4vondvuwdfbb/hive113001.png
  [2]: http://static.zybuluo.com/Great-Chinese/qdililvjbkjvxwrotvrbi8ve/hive113002.png
  [3]: http://static.zybuluo.com/Great-Chinese/xoy2wreezfetq2f95hkgt9ip/hive113003.png
  [4]: http://static.zybuluo.com/Great-Chinese/jjsb0jm0oy0bm9weumvtzxpo/hive_external.png
  [5]: http://static.zybuluo.com/Great-Chinese/t0ga0d8l735w3z4gtckmfguc/hvie_dop_table.png
  [6]: http://static.zybuluo.com/Great-Chinese/26nz2t47whjvdrr1v9d6yxs8/hive_drop_table_02.png
  [7]: http://static.zybuluo.com/Great-Chinese/g1vjif7fl1l6g9pf7loerxnw/hiveserver2.png
  [8]: http://static.zybuluo.com/Great-Chinese/a2ugr0277itch892zwbipkis/beeline.png
  [9]: http://static.zybuluo.com/Great-Chinese/d3nw4r0m0l3wbwakm3ejn1ct/beeline2.png