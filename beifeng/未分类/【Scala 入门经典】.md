# 【Scala 入门经典】

标签（空格分隔）： 未分类

[TOC]

---
#抽象类、抽象字段、抽象方法
```
/ScalaInAction/src/com/dt/scala/oop/AbstractClassOps.scala

package com.dt.scala.oop
class AbstractClassOps{
    var id : Int = _
}
abstract class SuperTeacher(val name : String){
  var id : Int
  var age : Int
  def teach
}
class TeacherForMaths(name : String) extends SuperTeacher(name){
  override var id = name.hashCode()
  override var age = 29
  override def teach{
    println("Teaching!!!")
  }
}
object AbstractClassOps{
  def main(args: Array[String]) {
      val teacher = new TeacherForMaths("Spark")
      teacher.teach
      println("teacher.id" + ":" + teacher.id)
      println(teacher.name + ":" + teacher.age)
  }
}
```
---
如果要想声明一个类为抽象类，必须用abstract关键字，这和java是一样的。在抽象类中的方法，不能写方法的实现体，也不需要添加abstract关键字，字段设置抽象时不能给值。
```
abstract class SuperTeacher(val name : String){
  var id : Int
  var age : Int
  def teach
}
```
如果一个类不是抽象类的话，字段必须赋值,没有值 给占位符
```
class AbstractClassOps{
    var id : Int = _
}
```
我们使用占位符只有一种情况var，val不可以使用占位符。

---
定义抽象类的子类
```
class TeacherForMaths(name : String) extends SuperTeacher(name){
  override var id = name.hashCode()
  override var age = 29
  override def teach{
    println("Teaching!!!")
  }
}
```
这里子类的name成员传给父类name成员

要复写抽象类的字段或方法可以用override，也可以不用，惯例加上。

---
```
object AbstractClassOps{
  def main(args: Array[String]) {
      val teacher = new TeacherForMaths("Spark")
      teacher.teach
      println("teacher.id" + ":" + teacher.id)
      println(teacher.name + ":" + teacher.age)
  }
}
```
运行结果如下：
```
Teaching!!!
teacher.id:80085693
Spark:29
```



