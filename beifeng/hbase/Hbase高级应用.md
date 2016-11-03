# Hbase高级应用

标签（空格分隔）： hbase

[toc]

---

# 1、Hbase支持压缩配置
##1.1创建$HBASE_HOME/lib/native/ 目录
##1.2创建软连接只想hadoop的native压缩库
```
ln -s ${HADOOP_HOME}/lib/native/ ${HBASE_HOME}/lib/native/Linux-amd64-64
```
1.2.1注意：Linux-amd64-64文件名说明
这个文件名你运行的操作系统还有hbase的版本有关，具体要查看hbase启动日志，Hbase启动时会打印系统属性日志: 
> zookeeper.ZooKeeper: Client environment:os.name=Linux
zookeeper.ZooKeeper: Client environment:os.arch=amd64

文件名最后一个64，是你操作系统的位数
1.2.2另外一种方式：
写代码获取系统这os.name和os.arch这两个参数，java代码写法如下：
```
public class GetSysPro {
	public static void main(String[] args) {
		System.out.println( "os.name=" + System.getProperty("os.name"));
		System.out.println( "os.arch=" + System.getProperty("os.arch"));
	}
}
```
javac GetSysPro.java
java GetSysPro
运行结果如下：
![java-get-pro.png-3.4kB][1]
##1.3配置完成后测试是否配置成功
> bin/hbase org.apache.hadoop.util.NativeLibraryChecker

看到如图结果，说明配置成功，配置完成后记得重启Hbase的Master和RegionServer服务
![ys.png-18kB][2]

#2.连接Hive
##2.1拷贝jar包到hivelib目录下
```
export HBASE_HOME=/opt/soft/cdh5/hbase-0.98.6-cdh5.3.6
export HIVE_HOME=/opt/soft/cdh5/hive-0.13.1-cdh5.3.6
cp $HBASE_HOME/lib/hbase-server-0.98.6-cdh5.3.6.jar $HIVE_HOME/lib/
cp $HBASE_HOME/lib/hbase-client-0.98.6-cdh5.3.6.jar $HIVE_HOME/lib/
cp $HBASE_HOME/lib/hbase-protocol-0.98.6-cdh5.3.6.jar $HIVE_HOME/lib/
cp $HBASE_HOME/lib/hbase-it-0.98.6-cdh5.3.6.jar $HIVE_HOME/lib/
cp $HBASE_HOME/lib/hbase-hadoop2-compat-0.98.6-cdh5.3.6.jar $HIVE_HOME/lib/
cp $HBASE_HOME/lib/high-scale-lib-1.1.1.jar $HIVE_HOME/lib/
cp $HBASE_HOME/lib/hbase-common-0.98.6-cdh5.3.6.jar $HIVE_HOME/lib/
cp $HBASE_HOME/lib/htrace-core-2.04.jar $HIVE_HOME/lib/
cp $HBASE_HOME/lib/hbase-hadoop-compat-0.98.6-cdh5.3.6.jar $HIVE_HOME/lib/
cp $HBASE_HOME/lib/htrace-core-2.04.jar $HIVE_HOME/lib/
```
##2.2配置conf/hive-site.xml，指定hbase zookeeper,不要设置端口号
```
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>test.sys.kuaixiuge.com</value>
</property>
```
##2.3Hive中创建表，查询Hbase表数据
```
#创建hive 在hbase上的内部表，则该表在hbase不能存在
CREATE TABLE xyz(id int, name string) 
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,info:name")
TBLPROPERTIES ("hbase.table.name" = "xyz");
#创建hive在hbase上的外部表，则该表必需在hbase上有
CREATE EXTERNAL TABLE xyz(id int, name string) 
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,info:name")
TBLPROPERTIES ("hbase.table.name" = "xyz");
```
##2.4创建分区表的几种方式
###2.4.1第一种
```
create 'split01' , 'info' , SPLITS=>[10,20,30,40,50]
```
###2.4.2第二种常用
首先准备文件/opt/txt/split.inp，每个分区key占一行
```
10
20
30
40
50
```
```
create 'split01', 'info', SPLITS_FILE => '/opt/txt/split.inp'
```
###2.4.3第三种方法 采用16进制进行分区，分区数为15个,不建议采用
```
create 'split03', 'info', {	VERSION => 3 , NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'}
```
##2.5修改表属性
```
#查看表属性
describe 'split02'
#修改表属性，这一定要加上列簇去修改，即 name的值才能修改完成
alter 'split02',  NAME => 'info', VERSIONS => 5 , COMPRESSION => 'SNAPPY'
```


  [1]: http://static.zybuluo.com/Great-Chinese/ieptqkzuefniqk6kl0mvujwy/java-get-pro.png
  [2]: http://static.zybuluo.com/Great-Chinese/w6h0tejcosajc08zal3j7l4a/ys.png