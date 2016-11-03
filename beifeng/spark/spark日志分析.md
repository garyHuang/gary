# spark日志分析

标签（空格分隔）： spark

[toc]

---

##1、spark读取日志分析
开发工具辅助类：
```
package com.bigdata.utils
object Helper {
  /*产生一个随机数*/
  def random(max:Int):Int={
    var id = (math.random * max).toInt  
    return id;
  }
  /*创建一个全数字的正则表达式*/
  val PARTTERN_INT = "^\\d+$".r
  /*将对象转换成为int对象*/
  def toInt(args:Object):Int={
    if(null != args){
      if(!PARTTERN_INT.findFirstMatchIn(args.toString()).isEmpty){
        return args.toString().toInt
      }
    }
    0
  }
  /*将对象转换成为Long对象*/
   def toLong(args:Object):Long={
    if(null != args){
      if(!PARTTERN_INT.findFirstMatchIn(args.toString()).isEmpty){
        return args.toString().toLong
      }
    }
    0
  }
}
```
