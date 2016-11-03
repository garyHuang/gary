# spark高级应用

标签（空格分隔）： spark

[toc]

---

#1.) spark wordcount程序
```
package org.gary

import org.apache.spark._

object WCScala {
  
  def main(args: Array[String]): Unit = {
      var fileName:String = "hdfs://cdh5.hadoop.com/opt/txt/wc.inp" ;
      var outputFile = "hdfs://cdh5.hadoop.com/opt/out/out" + random( 500 ) ;
      if(null != args && args.length > 0 ){
        fileName = args(0) 
        outputFile = args(1)
      }
      val conf = SparkCf.getSparkConf()
      
      var sc = new SparkContext( conf ) ;
      
      val rdd = sc.textFile( fileName )
      
      var  keyValueRdd = rdd.flatMap(_.split(" ")); 
      
      var wordRdd = keyValueRdd.map { x => (x , 1) } ;
       
       
      var result = wordRdd.reduceByKey(_ + _) ; 
      
      result.map( x => (x._2 , x._1)).sortByKey(false).map( x => (x._2 , x._1) )
      .saveAsTextFile( outputFile ) ; 
      /**
       * reduceByKey 会在Map Shuffle过程合一次，在reduce 端合并一次，相对减轻服务器的负担
       * groupBy 会全部在reduce端进行合并，相对来说增加服务器的负担
       *  所以 能用 reduceByKey 的一定要使用 reduceByKey
       * */
      /*停止SparkContext,释放资源 */
      sc.stop() ; 
  }
  
  def random(max:Int):Int={
    var id = (math.random * max).toInt  
    return id;
  } 
  
}
```

spark-submit提交任务
```
bin/spark-submit --master spark://cdh5.hadoop.com:7077 --class org.gary.WCScala lib/wc001.jar 
```




