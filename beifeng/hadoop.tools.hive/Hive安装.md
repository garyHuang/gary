# Hive安装

标签（空格分隔）： hadoop.tools.hive

[toc]

---
##1、下载Hive安装包
 下载地址：http://archive.apache.org/dist/hive/ 
 可以选择自己需要的hive版本，相对比较稳定的版本是0.1.3.1，本文使用该版本讲解

##2、安装Hive
###2.1 解压Hive
> tar -zxvf apache-hive-0.13.1-bin.tar.gz -C ./modules/

###2.2 修改文件名
```
    hive-default.xml.template --> hive-site.xml
    hive-env.sh.template  -->hive-env.sh
    hive-exec-log4j.properties.template -->  hive-exec-log4j.properties
    hive-log4j.properties.template -->  hive-log4j.properties
```
###2.3修改 hive-env.sh文件
```
    HADOOP_HOME=${HADOOP_HOME}
```

##3、启动测试Hive
```
  //进入HIVE_HOME目录执行命令
  bin/hive
```
![hive.png-7.7kB][1]

 查看hive的数据
 ```
 show databases;
 ```
 ![hive01.png-4.9kB][2]
 使用当前数据库
 ```
  use default;
 ```
![hive use databases.png-2.5kB][3]
创建表，表分隔符是TAB字符，两个字段。id int类型，name String类型
```
create table student(id int, name string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' ;
```
![hive createTable.png-3.7kB][4]
 准备学生文件 /opt/txt/student.txt
 内容如下：
```
1	黄飞
2	李四
3	王五
6	张三
```
###加载数据到hive中
```
load data inpath '/opt/txt/student.txt' into table student ;
```
![load tablt into hive.png-5.4kB][5]
###查询表中的数据
```
select * from student ;
```
![selectAllStudentInfo.png-3.9kB][6]
###只查询id一个字段

![hive-query-test.jpg-101.8kB][7]

###使用hive执行一个本地脚本sql
```
bin/hive -f /opt/txt/t.sql
/*
t.sql文件事先准备好，文件内容如下：
use default; 
select * from student ; 
*/
```
![hive-local-sql.png-10.3kB][8]
###将结果输出到一个文件里面
```
bin/hive -f /opt/txt/t.sql -> /opt/txt/out.txt
```
![hive-local-sql-output-local.png-8.8kB][9]
###查看输出文件内容
![hive-local-sql-output-local-catfile.png-5kB][10]


  [1]: http://static.zybuluo.com/Great-Chinese/46151jcu8v25re206m0iv9q9/hive.png
  [2]: http://static.zybuluo.com/Great-Chinese/fwk0ks0j8cj9kaejg0uo7byj/hive01.png
  [3]: http://static.zybuluo.com/Great-Chinese/ulagmoatu9nssivlgckqhnmz/hive%20use%20databases.png
  [4]: http://static.zybuluo.com/Great-Chinese/n8e1x055xd60umuia87fy2jr/hive%20createTable.png
  [5]: http://static.zybuluo.com/Great-Chinese/v78797lirpki7f3xbihdeu3d/load%20tablt%20into%20hive.png
  [6]: http://static.zybuluo.com/Great-Chinese/hx656s2ab20wew80k3xp6ik3/selectAllStudentInfo.png
  [7]: http://static.zybuluo.com/Great-Chinese/4e891gvzhbdpcsd1a2qmhvvf/hive-query-test.jpg
  [8]: http://static.zybuluo.com/Great-Chinese/nwp3v516phc8204w3olhgtik/hive-local-sql.png
  [9]: http://static.zybuluo.com/Great-Chinese/k6in79vghuoalh62q6lvo4tj/hive-local-sql-output-local.png
  [10]: http://static.zybuluo.com/Great-Chinese/qtz6gdffgko9biyevvlr2trh/hive-local-sql-output-local-catfile.png