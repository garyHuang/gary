# hive高级应用（apache日志分析+自定义函数(UDF)+数据倾斜）

标签（空格分隔）： hadoop.tools.hive

[toc]

---
1、正则表达式工具： http://wpjam.qiniudn.com/tool/regexpal/

日志文件内容格式：
```
"27.38.5.159" "-" "31/Aug/2015:00:04:37 +0800" "GET /course/view.php?id=27 HTTP/1.1" "303" "440" - "http://www.ibeifeng.com/user.php?act=mycourse" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.63 Safari/537.36" "-" "learn.ibeifeng.com"
```

## 一、web服务器日志导入到hive表中
###1、创建表
```sql
create table IF NOT EXISTS gary_hive.bf_log_src (
	ip string , 
	tm1 string , 
	stime string , 
	url string , 
	tmp1 string , 
	tmp2 string , 
	referer string , 
	user_agent string , 
	forwarded_for stirng , 
	host string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
"input.regex"="(\"[^ ]*\") (\"[^ ]*\") (\"[^\"]*\") (\"[^\"]*\") (\"[^ ]*\") (\"[^ ]*\") (-|[^ ]*) (\"[^ ]*\") (\"[^\"]*\") (\"[^\"]*\") (\"[^\"]*\")"
)
STORED AS TEXTFILE;
```
语句说明：
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe' 选择采用正则表达式对日志文件进行解析
WITH SERDEPROPERTIES : 输入的正则表达式，每一个小括号表示一个字段
使用正则表达式工具进行验证表达式是否正确：
![hive_regxpal.png-24.5kB][1]
###2、导入日志到hive表中
```sql
load data local inpath '/opt/txt/logs/moodle.ibeifeng.access.log' overwrite  into table db_bf_log.tomcat_log ; 
```
###3、测试是否成功
```
select ip,tm1,stime,url,tmp1,tmp2,tmp3,referer,user_agent,forwarded_for,http_host from tomcat_log limit 1 ;
```
![hive_test_query.png-30.2kB][2]

###4、查询出来的数据中，有我们不需要的引号和不满足需求的日期格式，这时需要用自定义udf函数进行转换
####4.1自定义去掉引号的UDF函数
```java
package hadoop.gary.bigdata.udf;

import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;

public class TrimQuotesUDF extends UDF {

	Text output = new Text();
	
	public Text evaluate(Text str) {
		try {
			if(null == str){
				return output ;
			}
			output.set(str.toString().replace("\"", ""));
		} catch (Exception e) {
			throw new RuntimeException( e );
		}
		return output ; 
	}
	
	public static void main(String[] args) {
		System.out.println( new TrimQuotesUDF().evaluate(new Text("\"27.38.5.159\"")));
	}
	
}
```
####4.2自定义日期转换UDF函数
```java
package hadoop.gary.bigdata.udf;

import java.text.SimpleDateFormat;
import java.util.Locale;

import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;

public class DateTimeUDF extends UDF {
	
	// 31/Aug/2015:00:04:53 +0800 
	SimpleDateFormat inputFormat  = new SimpleDateFormat("dd/MMM/yyyy:HH:mm:ss" , Locale.ENGLISH) ; 
	

	SimpleDateFormat outputFormat  = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss") ;
	Text output = new Text();
	public Text evaluate(Text str) {
		try {
			if (null == str) {
				return output;
			}
			String dateStr = str.toString().replace("\"", "") ; 
			
			output.set(outputFormat.format(inputFormat.parse(dateStr)));
			
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
		return output;
	}
	
	public static void main(String[] args) {

		System.out.println( new DateTimeUDF().evaluate(new Text("\"31/Aug/2015:00:04:53 +0800\"")));
	}

}
```
####4.3打包jar上传到linux上/opt/txt/lib/hive.jar
####4.4将函数加入hive上下文中
```
add jar /opt/txt/lib/hive.jar;
 create temporary function gary_trimquotes as 'hadoop.gary.bigdata.udf.TrimQuotesUDF';
 
 create temporary function gary_time as 'hadoop.gary.bigdata.udf.DateTimeUDF';
```
![hive_add_tmp_function.png-14kB][3]
####4.5测试查询
```sql
 select gary_trimquotes(ip) ,gary_tim(stime)  from tomcat_log limit 1 ; 
```
![hive_use_udf_query.png-9.1kB][4]
###5、优化hive
Fetch Task
```
<property>
  <name>hive.fetch.task.conversion</name>
  <value>minimal</value>
  <description>
    Some select queries can be converted to single FETCH task minimizing latency.
    Currently the query should be single sourced not having any subquery and should not have
    any aggregations or distincts (which incurs RS), lateral views and joins.
    1. minimal : SELECT STAR, FILTER on partition columns, LIMIT only
    2. more    : SELECT, FILTER, LIMIT only (TABLESAMPLE, virtual columns)
  </description>
</property>
```
MR压缩设置
```
1.设置map输出压缩
hive.exec.compress.intermediate=true 
mapreduce.map.output.compress=true 
mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec
2.设置reduce输出压缩
hive.exec.compress.output
mapreduce.output.fileoutputformat.compress
mapreduce.output.fileoutputformat.compress.codec
```
并行性
```
hive.exec.parallel.thread.number=8
hive.exec.parallel=true;
```
JVM重用
```
mapreduce.job.jvm.numtasks=4
```
Reduce的数目
```
设置数目通过测试获得，测试各种数目根据正太分布图取最后结果。
mapreduce.job.reduces=0;
```
推测执行
```
hive.mapred.reduce.tasks.speculative.execution=true;
mapreduce.map.speculative=true;
mapreduce.reduce.speculative=true;
```
###6.数据倾斜
参考文档：http://www.cnblogs.com/ggjucheng/archive/2013/01/03/2842860.html
导致的原因：
1、join 其中一个表较小，但是key集中.会将数据分发到某一个或几个Reduce上的数据远高于平均值
2、group by 维度过小，某值的数量过多 ，处理某值的reduce灰常耗时 
3、Count Distinct 某特殊值过多，  处理此特殊值的reduce耗时

参考调优：
1、hive.map.aggr=true
设置 Map 端部分聚合，相当于Combiner

2、hive.groupby.skewindata=true
默认值是false，需要设置成true ;
		当设置为true时，会变成两个MapReduce ;
第一个MR JOb中，map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样出来的结果相同的Group By Key有可能被分发到不同的Reduce中，从而达到辅助均衡目的。
3、将特定的值，进行特定的处理。
   比如是null，
   * 过滤掉，where case
```
select * from log a
  join users b
  on a.user_id is not null
  and a.user_id = b.user_id
union all
select * from log a
  where a.user_id is null;
```
  * 特定方式转换特定的值，使得这些值不一样，同时这些值不影响分析。
```
select *
  from log a
  left outer join users b
  on case when a.user_id is null then concat(‘hive’,rand() ) else a.user_id end = b.user_id;
```
 *采用join时，尽量使用同一种数据类型，不同数据类型尽量转换成一致的数据类型。





  [1]: http://static.zybuluo.com/Great-Chinese/q73e3c02xe76scuobvsufgvu/hive_regxpal.png
  [2]: http://static.zybuluo.com/Great-Chinese/czs2ksm1q2puqb0ucz9gdzb8/hive_test_query.png
  [3]: http://static.zybuluo.com/Great-Chinese/ect8uaj5xmnimxc78z7hsdaa/hive_add_tmp_function.png
  [4]: http://static.zybuluo.com/Great-Chinese/a9hgm1yuu8a59qytzjioauo5/hive_use_udf_query.png