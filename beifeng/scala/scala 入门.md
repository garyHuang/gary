# scala 入门


标签（空格分隔）： scala

[toc]

---
API 中文查询文档：
http://www.yiibai.com/scala/

mvn创建scala项目
```
mvn archetype:generate -DarchetypeGroupId=net.alchim31.maven -DarchetypeArtifactId=scala-archetype-simple -DremoteRepositories=http://scala-tools.org/repo-releases -DgroupId=org.gary.spark -DartifactId=org.gary.spark -Dversion=1.0-SNAPSHOT
```

#1) def 修饰函数 var修饰变量
    def定义的变量，值无法更改，相当于一个函数，每次调用会自动初始化
    举例：
```
def array = Array( 1 ,3 , 4 , 5 ) ;
array(3) = 111 ;
println( array.mkString(",")) 
//输出值为： 1,3,4,5
var array = Array( 1 ,3 , 4 , 5 ) ;
array(3) = 111 ;
println( array.mkString(",")) 
//输出值为： 1,3,4,111
```
#2) scala数组操作和for循环
```
package org.gary.app

import scala.collection.mutable.ArrayBuffer;
import scala.util.Sorting;
object FunTest {
  /**
   * 计算最大值
   * 
   * 函数 从那数定义规则
   *   ( num1 : Long) : Long
   *   ( 变量名称  : 变量类型) : 返回类型
   * */
  def max(num1 : Long , num2 : Long): Long = {
     if(num1 > num2){
       num1
     }
     num2
  }
  /**
   * 测试使用函数
   * 定义无参数 无返回值的函数
   * */
  def testArray(){
     /*初始化数组，无法修改数组内容*/
    def items = Array("A" , "B")
    items(0) = "F";
    for( a <- items){
      println( a ) 
    }
    /*scala动态数组*/
    var arrayBuffer = ArrayBuffer[String](  );
    arrayBuffer += "Scala,";
    arrayBuffer += ("Hello" , "World");
    arrayBuffer ++= Array("Hello" , "World");
    arrayBuffer.insert(2, "Gary" , "Space");
   for( a <- arrayBuffer){
      print( a + " ") ;
    }
    println() ; 
    /* 固定数组长度，并修改值*/
    val greetStrings = new Array[String](3);
    greetStrings(0)="Hello";
    greetStrings(1) = "," ;
    greetStrings(2)="world\n" ;
    for(i <- 0 to 2){
    	print( greetStrings(i) );
    }
    /*定义一个数组*/
   var intArray =  Array(5,6,7,8,1,2,3,4);
   
  var result = for( i <- intArray) yield 2 * i
  //数组快熟排序
  Sorting.quickSort( result );
     for(i <- result){
    	print( i + " ");
     }
     println() ; 
     
     /*for(i <- 0 until result.length)
      * until 不计算最后一个数字
      * 
      * for(i <- 0 to result.length)
      * 计算最后一个数字
      * */
     for(i <- 0 until result.length){
    	print( result(i) + " " );
     }
      println
     
     println( result.mkString(" and ") )
     println( result.mkString("<" , "," , ">") )
  }
  
  /*
  scala Main 方法
  **/
  def main(args: Array[String]): Unit = {
    println( max(150 , 122)) ;
    testArray();
    
  }
}
```
#3）与java不同的私有变量
```
package org.gary.app

class Person {
  /*指定只能当前对象调用，*/
  private[this] var age = 0 ; 
  def data = age ;
  def add(){
    age += 1
  }
  def compare( person : Person ):Boolean={
    /*由于age使用 this关键字标识，这里用person.age就无法访问该变量，采用this.age 就可以正常访问*/
    return person.data == this.age ; 
  }
}
```
#4）构造器使用
```
package org.gary.app

class DefaultConstructor ( name:String , age:Int){
  def this(name:String){
    /*自定义构造器，必需首先调用默认构造器*/
    this(name , 24) ; 
  }
  def show(){
    println( name + "-->" + age ) ;
  }
}

Main方法调用代码
var const = new DefaultConstructor( "Gary" , 15) ;
const.show() ; 
 const = new DefaultConstructor( "Meloldy" ) ;
const.show(); 
```
#5)三目运算
> if (dataNum == 1)""else"1"

#6)case class 和 case object的使用
```
abstract class Person
/*当class设置为case时，实例对象可以访问构造器中的属性*/
case class Student(age:Int) extends Person
case class Worker(age:Int,sal:Double) extends Person
case object People extends Person
object CaseClass {
  def main(args: Array[String]): Unit = {
   def caseOps(person:Person) = person match{
     case Student(age) => println("I am " + age);
     case Worker(_ , sal) => println( "I got:" + sal );
     case People => println("People") ;
   }
   /*case一个class对象*/
   caseOps(Student(18)) ; 
   /*case一个Object对象*/
   caseOps( People ) ; 
   /*创建work对象*/
   var work1 = Worker(18 , 3500) ; 
   /*复制一个新的work对象，改变工资*/
   var work2 = work1.copy(sal = 160);
   /*复制一个新的work对象,改变年龄和工资*/
   var work3 = work1.copy(age = 25 , sal = 150);
   /*输出三个worker*/
   println( work1.age + "---" + work1.sal ) ; 
   println( work2.age + "---" + work2.sal ) ; 
   println( work3.age + "---" + work3.sal ) ; 
  }
}
```
#7) scala 让人摸不着头脑的apply
```
package org.oop

/**
 * 定义一个ApplyTest class 默认构造函数传递name:String
 * */
class ApplyTest(name:String){
  def this(name:String,age:Int){
    this(name)
  }
  def apply() = {
    println("class ApplyTest apply")
  }
  def haveATry{
    println("Have a try on apply!")
  }
}
/**
 * 定义一个ApplyTest object
 *  object 不能定义构造器
 * */
object ApplyTest{
  
  def apply() = {
    println("object ApplyTest apply")
    /*将 ApplyTest class 实例化，传递个当前对象,如果没有这个实例化，当前ApplyTest object的实例就不能调用 ApplyTest class类的方法*/
    new ApplyTest("Gary")
  }
}
object ApplyOperation {
  def main(args: Array[String]) {
    val a = new ApplyTest("Gary")
    /*默认调用 class ApplyTest 的haveATry函数*/
    a.haveATry
    /*默认调用 class ApplyTest 的apply函数*/
    a() 
    println( "----------------------------------" )
    /*默认调用 object ApplyTest 的apply函数*/
    val b = ApplyTest()
    /*调用 ApplyTest class 类中的 haveATry函数*/
    b.haveATry
    /*调用 ApplyTest class 类中的 apply函数*/
	  b()
    /*调用 ApplyTest class 类中的 apply函数*/
    b.apply();
  }
}
```
#8）sacla for循环和List，Map，Tuple的使用
```scala
/*定义一个空的List*/
var list = List[Int]( );
/*定义一个 默认值的List*/
var list2 = List[Int]( 11,12 );
/*循环在list中加入一个值*/
for(i <- 1 to 10){
  list = list:+i
}
/*在一个list中加入另外一个list*/
list = list.++(list2)
/*删除list中值小于3的数字*/
list = list.dropWhile { x => x <= 3 }
/*修改list中第一个索引元素的位置*/
list = list.updated(0 , 9)  
println( list )
println( list(2) )
```
#9sacla List排序
```scala
 var shuffledData = List( 6,2,5,76,8,2,2,3,4,5);
println( sortList(shuffledData) )
/*对集合进行排序*/
def sortList(list:List[Int]):List[Int]=list match{
  case List() => List()
  case head :: tail => compule(head ,sortList(tail)) ;
}

def compule(data:Int,dataSet:List[Int]):List[Int]=dataSet match{
  /*如果dataSet为空，直接返回data，放入List中*/
   case List() => List(data)
   /* 如果 dataSet 不为空，则比较 data和dataSet第一个元素的大小，如果大则放在前面
     ， 如果小 则放在前面，将dataSet第二个开始的元素进行排序*/
    case head :: tail => 
      if(data >= head){
          data::dataSet 
      }
      else{
         head :: compule( data , tail)
      }
}
```
#foldLeft，foldRight ，span,takeWhile
```
var testMkStringList = List( 1  , 3 , 2);
println( (1 to 10).foldLeft(100)(_-_) )  //结果是45
println( (1 to 10).foldRight(100)(_-_) )  //结果是95
/*foldLeft的另外一种写法*/
 println( ("" /: List("1","2" , "3")) (_+_) ) 
/*计算步骤是 
 * 1+2 = 12
 * 12+3=123
 * */
 
/*foldRight的另外一种写法*/
println( (List("1","2" , "3") :\ "")(_+_))
/*计算步骤
 * 2+3 = 23
 * 1+23 = 123
 * */
 

/*span 对list拆分，从第1一个开始，碰到第一个不小于2的数字开始进行拆分成为两个list*/ 
println( testMkStringList.span(_ < 2) ) /*结果为：(List(1),List(3, 2))*/
/*takeWhile 循环获取，从第一个开始，取到第一个不小于2的数字 数字开始提前函数*/
println(testMkStringList.takeWhile { x => x < 2 }); /*结果：List(1)*/
```
#8）scala 多重集成
```
/**
 * @author Administrator
 */
trait T1
class T2{
  /*这里self不是固定名字，可由自己喜好定义，常用 self和this或者
   * 将类名称首字母小写 作为该变量，指定返回类型为T1，而
   * 该类没有集成T1，说明这个类实例化的时候必需采用
   * with T1进行实例化
   * 实例代码：
   * 	new T2 with T1(); 
   * 这句话 必需写在类定义的第一行定义
   * */
  self:T1=>
  var temp = "T2" ;
  def foo = self.temp + "-->" + this.temp;
}

object TSExtends {
  def main(args: Array[String]): Unit = {
    var t2 = new T2 with T1(); 
    println( t2.foo )
  }
}
```
#9)隐式转换
```
import scala.math.Ordered
import scala.io.Source
/*A类属于Comparable的子类，不会进行隐式转换*/
class TComparableColon[A <: Comparable[A]](val f:A,var s:A){
  def bigger = {
    if(f.compareTo(s) > 0) f else s 
  } 
}
/*这里会将传入的参数，隐式的转换成为Comparable的子类*/
class TComparableSign[A <% Comparable[A]](val f:A,var s:A){
  def bigger = {
    if(f.compareTo(s) > 0) f else s
  }
}
/* 将比较对象，隐式转换成为Ordered对象，可以使用大于号小于号 进行 比较 传入的对象
 * 如果是String对象，这里会自动转换成为 RichString*/
class TComparableSignOrdered[A <% Ordered[A]](val f:A,var s:A){
  def bigger = {
     if(f > s) f else s
  }
}

/*第一种ordering隐式转换*/
class TComparableOrdering[T:Ordering](var f:T,var s:T){
   def bigger(implicit order:Ordering[T]) = {
     order.max(f, s)
  }
}
/*第二种ordering隐式转换*/
class TComparableOrderingToOrdered[T:Ordering](var f:T,var s:T){
   def bigger = {
     import Ordered._
     if(f > s ) f else s 
  }
}

/*第三种ordering隐式转换*/
class TComparableImplicitly[T:Ordering](var f:T,var s:T){
   def bigger = {
     if(implicitly[Ordering[T]].compare(f , s) > 0 )
      f else s
  }
}

class RichFile(file:java.io.File){
  def read = Source.fromFile(file).mkString
}
/*自定义隐式转换类，必需object定义*/
object Context{
  /*将java.io.File隐式转换成为RichRile对象，这个函数的参数只能是java.io.File对象
   * 返回对象为RichFile对象,当实例化 java.io.File对象时，系统就会自动调用这个函数，返回
   * RichFile对象*/
  implicit def file2RichFile(file:java.io.File) : RichFile 
  = new RichFile(file)
  /**
   * 给Int类型新增一个隐式转换类
   * */
  implicit class Op(x:Int){
    /*给Int数字新增一个add方法*/
    def add(s:Int) : Int = x + s ;
  }
}

object ContextParam{
  implicit var default:String = "java" ;
}

object Param{
  def printLog(s1:String)(implicit default:String):Unit = {
    println( default +":" + s1 ) 
  }
}
/**
 * 使用隐式转换类
 * */
object ImplicitFormat {
  def main(args: Array[String]): Unit = {
      var colon = new TComparableColon("Spark" , "Hadoop")
      println( "colon==" +  colon.bigger )
      /*这种方式不行,因为整数 3 和5是Int类型，不是 Comparable的子类*/
      //var colon1 = new TComparableColon(3, 5)
      //println( colon1.bigger )
      
      /*
       * 测试 隐式转换的使用方法
       * */
      var sign = new TComparableSign("Spark" , "Hadoop")
      println("sign==" +  sign.bigger )
      var sign1 = new TComparableSign(3 , 5 ) /*这里隐式的将Int,转换成为RichInt*/
      println( "sign1==" +  sign1.bigger )
      
      /*测试隐式转换城Ordered对象进行比较对象大小*/
      var signOrdered = new TComparableSignOrdered("Spark" , "Hadoop") ; /*String会隐式转换成为RichString*/
      println(  "signOrdered==" +  signOrdered.bigger );
      
      var signOrdered1 = new TComparableSignOrdered( 3 , 6) ;
      println( "signOrdered1==" + signOrdered1.bigger );
      
       /*测试隐式转换城Ordered对象进行比较对象大小*/
      var ordering = new TComparableOrdering("Spark" , "Hadoop") ; /*String会隐式转换成为RichString*/
      println( "ordering===" + ordering.bigger );
      println( "--------------------------" );
      /*导入隐式转换，将file转换成为RichFile*/
      import Context._
      /* 实例化java.io.File对象，自动调用 Context.file2RichFile方法，
       * 将对象转换成功RichFile对象*/
      println( new java.io.File("D:/1.txt").read);
      
      /*整数本来没有add方法，通过隐式转换 给整数新增一个add方法*/
      println(1.add( 2 ));
      
      println( "--------------------------" );
      /*隐式参数调用，当没有导入隐式参数转换的时候，调用下面函数必需，是两个参数*/
      //Param.printLog("hadoop" ) /*这个语句是便宜不通过,因为没有导入隐式参数函数*/
      Param.printLog("hadoop" )( "Scala")
      /*这里导入ContentParam的所有隐式参数*/
      import ContextParam.default
      /*再次调用该函数，就可以只传递一个参数，第二各参数默认为ContextParam.default的值java*/
      Param.printLog("hadoop")
      /*如果传递Spark，就会覆盖原来的java值*/
      Param.printLog("hadoop")("Spark")
  }
}
```
##9.10)scala读取文件
```
package com.dt.scala.oop3

import scala.io.BufferedSource
import scala.io.Source
import java.net.InetAddress

 
trait Reader {
  type In <: java.io.Serializable;
  type ReturnType
  def read(in:In) : ReturnType 
}

class FileReader extends Reader{
  type In = String 
  type ReturnType = BufferedSource
  
   def read(in:In) : ReturnType = {
     Source.fromFile( in , "GBK")
   }
}
/*
Exception in thread "main" java.nio.charset.MalformedInputException: Input length = 1
如包读取文件报错这个信息，一定是字符编码格式不匹配，需要查看文件字符编码，
然后读取文件代码为：
	Source.fromFile( in , "GBK") 
 */
object TestFileReader {
  def main(args: Array[String]): Unit = {
    
    /*
     val reader = new FileReader
    var txt = reader.read( "D:\\1.txt" ) ;
    for(  line <- txt.getLines ){
      println(line)
    }
     * */
    
    val reader = new FileReader
    var txt = reader.read( "D:\\1.txt" ) ;
    var is = txt.reader()
    
    var chars = new Array[Char]( 15 )
    var len = -1 
    len = is.read(chars, 0 , chars.length)
    var buffer = new StringBuilder;
    while( len != -1 ){
      buffer.append( new String(chars , 0 , len)) 
      len = is.read( chars , 0 , chars.length)
    }
    println( buffer)
    
  }
}
```
