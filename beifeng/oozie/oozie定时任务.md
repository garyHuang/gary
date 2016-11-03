# oozie定时任务

标签（空格分隔）： oozie

[TOC]

---
#1) 创建 hive_gary.track_log表
```
drop table if exists hive_gary.track_log ;
create table hive_gary.track_log (
id                         string ,
url                        string ,
referer                    string ,
keyword                    string ,
type                       string ,
guid                       string ,
pageId                     string ,
moduleId                   string ,
linkId                     string ,
attachedInfo               string ,
sessionId                  string ,
trackerU                   string ,
trackerType                string ,
ip                         string ,
trackerSrc                 string ,
cookie                     string ,
orderCode                  string ,
trackTime                  string ,
endUserId                  string ,
firstLink                  string ,
sessionViewNo              string ,
productId                  string ,
curMerchantId              string ,
provinceId                 string ,
cityId                     string ,
fee                        string ,
edmActivity                string ,
edmEmail                   string ,
edmJobId                   string ,
ieVersion                  string ,
platform                   string ,
internalKeyword            string ,
resultSum                  string ,
currentPage                string ,
linkPosition               string ,
buttonPosition             string
) 
PARTITIONED BY (date string,hour string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' ;
```
#2） Shell 脚本上传文件到hdfs 创建文件 load_data.sh
```
#!/bin/sh

## loading system enviroment variable
. /etc/profile

## set hive home
HIVE_HOME=/opt/soft/cdh5/hive-0.13.1-cdh5.3.6


## get yesterday date
#yesterday=`date -d -1days '+%Y%m%d'`

## set logs directory
LOG_DIR=/opt/txt/
#循环文件夹 的文件 自动加载到hive中
for line in `ls $LOG_DIR`
do
    #### echo $line
	## get date and hour
	date=${line:0:4}${line:4:2}${line:6:2}
	hour=${line:8:2}
	
	###echo $date $hour
	$HIVE_HOME/bin/hive -e "load data local inpath '$LOG_DIR/$line' overwrite into table hive_gary.track_log partition(date = '$date', hour = '$hour') ;"
done
```
#3) 创建表 daily_hour_visit，并解析分析 log日志， 创建文件 hive-parse.sql
```
use hive_gary ;

drop table if exists daily_hour_visit;

create table if not exists daily_hour_visit (
date                         string ,
hour                        string ,
pv                    string ,
uv                    string
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' ;


insert into table hive_gary.daily_hour_visit
select 
  date, hour, count(url) pv, count(distinct guid) uv
from hive_gary.track_log main
where not exists(
select 1 from hive_gary.daily_hour_visit sub where sub.date = 
main.date and sub.hour = main.hour
)
group by date, hour ; 
```
#4）使用sqoop将日志导入到mysql中
```
export --connect jdbc:mysql://${hostName}:3306/test --username root --password 123456 --table daily_hour_visit --num-mappers 1 --input-fields-terminated-by "\t" --export-dir ${nameNode}/user/hive/warehouse/hive_gary.db/daily_hour_visit/
```

#5）创建workflow.xml
```

<workflow-app xmlns="uri:oozie:workflow:0.2" name="wf-visit">
	
	<start to="shell-node"/>
	<!-- shell Action -->
    <action name="shell-node">
        <shell xmlns="uri:oozie:shell-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <exec>${EXEC}</exec>
			<file>${nameNode}/${oozieAppsRoot}/wf-visit/${EXEC}#${EXEC}</file>
            <capture-output/>
        </shell>
        <ok to="hive-node"/>
        <error to="shell-fail"/>
    </action>
	<!-- hive Action -->
	<action name="hive-node">
        <hive xmlns="uri:oozie:hive-action:0.5">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
			<job-xml>${nameNode}/${oozieAppsRoot}/wf-visit/hive-site.xml</job-xml>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <script>hive-parse.sql</script>
        </hive>
        <ok to="sqoop-node"/>
        <error to="hive-fail"/>
    </action>
	<!-- sqoop Action -->
	<action name="sqoop-node">
        <sqoop xmlns="uri:oozie:sqoop-action:0.3">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <command>export --connect jdbc:mysql://${hostName}:3306/test --username root --password 123456 --table daily_hour_visit --num-mappers 1 --input-fields-terminated-by "\t" --export-dir ${nameNode}/user/hive/warehouse/hive_gary.db/daily_hour_visit/</command>
        </sqoop>
        <ok to="end"/>
        <error to="sqoop-fail"/>
    </action>
	
	
	<kill name="shell-fail">
        <message>Shell action failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>	
	
	<kill name="hive-fail">
        <message>Hive action failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
	
	<kill name="sqoop-fail">
        <message>Sqoop failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
	
    <end name="end"/>
</workflow-app>

```
#6）创建 job.properties
```
hostName=cdh5.hadoop.com
nameNode=hdfs://${hostName}:8020
jobTracker=${hostName}:8032
queueName=default

oozieAppsRoot=user/gary/gywf
inputDir=opt/data
outputDir=opt/out/out6
# shell文件名称
EXEC=load_data.sh
#本地log文件的路径
log_path=/opt/txt
##oozie自动扫描包
oozie.use.system.libpath=true
##workflow.xml的路径
#oozie.wf.application.path=${nameNode}/${oozieAppsRoot}/wordcount/workflow.xml
##配置定时任务开始时间
start=2016-1-03T17:11+0800
##配置定时任务结束时间
end=3016-01-01T23:30+0800
##配置 workflow 定时任务在hdfs上的路径
workflowAppUri=${nameNode}/${oozieAppsRoot}/wf-visit/
##workflow的位置
oozie.coord.application.path=${workflowAppUri}

```
#7)创建 coordinator.xml文件
```

<!-- frequency="${coord:minutes(1)}" 每分钟执行依次
frequency="00 23 * * *" 每天23点整执行一次
-->
<coordinator-app name="cron-coord" frequency="00 23 * * *" start="${start}" end="${end}" timezone="GMT+0800"
                 xmlns="uri:oozie:coordinator:0.4">
    <action>
        <workflow>
            <app-path>${workflowAppUri}</app-path>
            <configuration>
                <property>
                    <name>jobTracker</name>
                    <value>${jobTracker}</value>
                </property>
                <property>
                    <name>nameNode</name>
                    <value>${nameNode}</value>
                </property>
                <property>
                    <name>queueName</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
        </workflow>
    </action>
</coordinator-app>

```

#8)运行
> bin/oozie job -oozie http://cdh5.hadoop.com:11000/oozie -config gywf/wf-visit/job.properties -run

#9）运行完毕后查看mysql数据库表
![oozie-corn.png-11.1kB][1]


  [1]: http://static.zybuluo.com/Great-Chinese/0w18muyae9qf7toj0e2hqhi5/oozie-corn.png