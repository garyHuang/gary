# oozie安装

标签（空格分隔）： oozie

[TOC]

---
#1)下载
http://archive.cloudera.com/cdh5/cdh/5/oozie/DG_QuickStart.html
并下载extjs
http://dev.sencha.com/deploy/ext-2.2.zip

#2)安装
 解压 oozie-hadooplibs-4.0.0-cdh5.3.6.tar.gz 到上一级目录
 ```
 tar -zxvf oozie-hadooplibs-4.0.0-cdh5.3.6.tar.gz -C ../
 ```
创建 libext 目录
> mkdir libext

拷贝对应的jar包到libext目录
> cp hadooplibs/hadooplib-2.5.0-cdh5.3.6.oozie-4.0.0-cdh5.3.6/* libext

拷贝下载extjs.zip 到libext目录

运行命令三个命令
a) bin/oozie-setup.sh prepare-war
b) bin/oozie-setup.sh sharelib create -fs hdfs://cdh5.hadoop.com:8020 -locallib oozie-sharelib-4.0.0-cdh5.3.6-yarn.tar.gz
c)bin/ooziedb.sh create -sqlfile oozie.sql -run DB Connection
d) bin/oozied.sh start 
e)解压 oozie-examples.tar.gz 得到 examples目录，并上传到hdfs当前用户的根目录
bin/hadoop dfs -put examples/ examples

#3) 修改examples/apps/map-reduce/job.properties 文件
```
nameNode=hdfs://cdh5.hadoop.com:8020
#这里很重要，端口一定要是 8032提交到yarn上运行
jobTracker=cdh5.hadoop.com:8032
queueName=default
examplesRoot=examples
oozie.wf.application.path=${nameNode}/user/${user.name}/${examplesRoot}/apps/map-reduce/workflow.xml
outputDir=map-reduce
```
运行一个任务 
oozie job -oozie http://cdh5.hadoop.com:11000/oozie -config examples/apps/map-reduce/job.properties -run

#3)配置oozie连接mysql
在conf/oozie-site.xml 文件中找到以下几个属性，并修改为为mysql地址
拷贝mysql驱动包到libext目录下

```
<property>
        <name>oozie.service.JPAService.jdbc.driver</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
        <name>oozie.service.JPAService.jdbc.url</name>
        <value>jdbc:mysql://cdh5.hadoop.com/oozie?createDatabaseIfNotExist=true</value>
    </property>

    <property>
        <name>oozie.service.JPAService.jdbc.username</name>
        <value>root</value>
    </property>

    <property>
        <name>oozie.service.JPAService.jdbc.password</name>
        <value>123456</value>
    </property>
```
由于mysql上没有oozie的初始化sql，所以需要初始化，运行命令
bin/ooziedb.sh create -sqlfile oozie.sql -run DB Connection

初始化完成后，如果立即启动oozie系统就会报错找不到mysql驱动“com.mysql.jdbc.Driver”，
由于oozie启动 类似tomcat启动，这里需要拷贝一个mysql驱动到oozie项目的lib目录下面，
oozie-server/webapps/oozie/WEB-INF/lib/
拷贝后启动oozie
oozied.sh start 访问就不会出现错误了

测试一下oozie坏境 
```
bin/oozie job -oozie http://cdh5.hadoop.com:11000/oozie -config examples/apps/map-reduce/job.properties -run
```
看到状态为SUCCEEDED,说明已经配置成功
![oozied.png-51.4kB][1]
再查看mysql下是否多了oozie的任务数据
![oozied-mysql.png-34.8kB][2]

#4）测试一个workflow
准备一个 workflow.xml，输入内容为
```

<workflow-app xmlns="uri:oozie:workflow:0.2" name="map-wd-wf">
    <start to="mr-node"/>
    <action name="mr-node">
        <map-reduce>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <prepare>
                <delete path="${nameNode}/${outputDir}"/>
            </prepare>
            <configuration>
				
				
                <property>
                    <name>mapred.mapper.new-api</name>
                    <value>true</value>
                </property>
				
				<property>
                    <name>mapred.reducer.new-api</name>
                    <value>true</value>
                </property>
				
				<property>
                    <name>mapreduce.job.map.class</name>
                    <value>hadoop.gary.bigdata.mapreduce.WordCountMapReduce$WordCountMapper</value>
                </property>
				<property>
                    <name>mapreduce.map.output.key.class</name>
                    <value>org.apache.hadoop.io.Text</value>
                </property>
				
				<property>
                    <name>mapreduce.map.output.value.class</name>
                    <value>org.apache.hadoop.io.LongWritable</value>
                </property>
                <!-- 设置reduce相关 -->
                <property>
                    <name>mapreduce.job.reduce.class</name>
                    <value>hadoop.gary.bigdata.mapreduce.WordCountMapReduce$WordCountReduce</value>
                </property>
				<property>
                    <name>mapreduce.job.output.key.class</name>
                    <value>org.apache.hadoop.io.Text</value>
                </property>
				<property>
                    <name>mapreduce.job.output.value.class</name>
                    <value>org.apache.hadoop.io.LongWritable</value>
                </property>
				
				<!-- 设置输入目录 -->
                <property>
                    <name>mapred.input.dir</name>
                    <value>${nameNode}/${inputDir}/</value>
                </property>
				<!-- 设置输出目录 -->
                <property>
                    <name>mapred.output.dir</name>
                    <value>${nameNode}/${outputDir}</value>
                </property>
            </configuration>
        </map-reduce>
        <ok to="end"/>
        <error to="fail"/>
    </action>
    <kill name="fail">
        <message>Map/Reduce failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name="end"/>
</workflow-app>

```
准备job.properties
```

nameNode=hdfs://cdh5.hadoop.com:8020
jobTracker=cdh5.hadoop.com:8032
inputDir=opt/data
outputDir=opt/out/out6
queueName=default
oozieAppsRoot=user/gary/gywf
# workflow 文件在hdfs上的路径
oozie.wf.application.path=${nameNode}/${oozieAppsRoot}/wordcount/workflow.xml

```
使用命令强制上传文件
> ../hadoop-2.5.0-cdh5.3.6/bin/hdfs dfs -put -f -p gywf /user/gary/

运行该程序
> bin/oozie job -oozie http://cdh5.hadoop.com:11000/oozie -config gywf/wordcount/job.properties -run

测试对比下，两个程序生成的结果是否正确

#5)运行hive workflow.xml
准备 workflow.xml
```
<?xml version="1.0" encoding="UTF-8"?>

<workflow-app xmlns="uri:oozie:workflow:0.5" name="hive-wf">
    <start to="hive-node"/>

    <action name="hive-node">
        <hive xmlns="uri:oozie:hive-action:0.5">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <prepare>
                <delete path="${nameNode}/${outputDir}"/>
            </prepare>
			<job-xml>${nameNode}/${oozieAppsRoot}/hive-select/hive-site.xml</job-xml>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <script>hive-select.sql</script>
            <param>OUTPUT=${nameNode}/${outputDir}</param>
        </hive>
        <ok to="end"/>
        <error to="fail"/>
    </action>

    <kill name="fail">
        <message>Hive failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name="end"/>
</workflow-app>

```

准备job.properties
```
nameNode=hdfs://cdh5.hadoop.com:8020
jobTracker=cdh5.hadoop.com:8032
queueName=default
oozieAppsRoot=user/gary/gywf
oozie.use.system.libpath=true
inputDir=opt/data
outputDir=opt/hive/out
oozie.wf.application.path=${nameNode}/${oozieAppsRoot}/hive-select
```
sql文件
```
insert overwrite directory '${OUTPUT}' SELECT pid , COUNT(1) FROM hive_gary.epc_partgroup GROUP BY pid ;
```
创建lib目录，放如mysqljar包
目录结构为：
![hive-oozie.png-4.6kB][3]
开启任务：
> bin/oozie job -oozie http://cdh5.hadoop.com:11000/oozie -config gywf/hive-select/job.properties -run

去hdfs上查看 /opt/hive/out/0* 的内容。

查看oozie运行是日志
tail -200f logs/oozie.log

#6）oozie时区配置
修改conf/oozie-site.xml
```
<property>
        <name>oozie.processing.timezone</name>
        <value>GMT+0800</value>
    </property>
```
修改oozie-server/webapps/oozie/oozie-console.js 177行代码
```
function getTimeZone() {
    Ext.state.Manager.setProvider(new Ext.state.CookieProvider());
    return Ext.state.Manager.get("TimezoneId","GMT+0800");
}
```
在getTimeZone下创建方法：
```
function getTimeZoneStr() {
    Ext.state.Manager.setProvider(new Ext.state.CookieProvider());
    return "GMT%2B0800";
}
```
修改oozie-console.js的524行:
原代码：
> url:getOozieBase() + 'job/' + workflowId + "?timezone=" + getTimeZone() 

修改为：
> url:getOozieBase() + 'job/' + workflowId + "?timezone=" + getTimeZoneStr()

修改oozie-console.js的1685行:
原代码：
> url: getOozieBase() + 'job/' + workflowId + "?timezone=" + getTimeZone()

修改为：
> url: getOozieBase() + 'job/' + workflowId + "?timezone=" + getTimeZoneStr()

#7)oozie运行sqoop任务
workflow.xml文件内容
```
<?xml version="1.0" encoding="UTF-8"?>
<workflow-app xmlns="uri:oozie:workflow:0.1" name="sqoop-export-wf">
    <start to="sqoop-node"/>
    <action name="sqoop-node">
        <sqoop xmlns="uri:oozie:sqoop-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
				
            </configuration>
            <command>export --connect jdbc:mysql://cdh5.hadoop.com:3306/test --username root --password 123456 --table new_part --num-mappers 1 --input-fields-terminated-by "\t" --export-dir ${nameNode}/user/hive/warehouse/hive_gary.db/partgroup/</command>
            <!-- 这里一定要是绝对路径，不能是相对路径 -->
        </sqoop>
        <ok to="end"/>
        <error to="fail"/>
    </action>

    <kill name="fail">
        <message>Sqoop failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name="end"/>
</workflow-app>
```
job.properties
```
nameNode=hdfs://cdh5.hadoop.com:8020
jobTracker=cdh5.hadoop.com:8032
queueName=default
oozieAppsRoot=user/gary/gywf
oozie.use.system.libpath=true
inputDir=opt/data
outputDir=opt/hive/out
oozie.wf.application.path=${nameNode}/${oozieAppsRoot}/hive-select

```
运行任务
> bin/oozie job -oozie http://cdh5.hadoop.com:11000/oozie -config gywf/sqoop-export/job.properties -run

导入mysql表中，如果发现乱码，则修改mysql字符集 为UTF8



  [1]: http://static.zybuluo.com/Great-Chinese/h8212x95blj2ltjf5yciagli/oozied.png
  [2]: http://static.zybuluo.com/Great-Chinese/3h60tmhxxb0opa4rekphpahi/oozied-mysql.png
  [3]: http://static.zybuluo.com/Great-Chinese/qyrkf7hzba2sd7z8vt6gd71x/hive-oozie.png