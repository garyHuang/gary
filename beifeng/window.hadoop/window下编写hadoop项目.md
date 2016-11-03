# window下编写hadoop项目

标签（空格分隔）： window.hadoop

---

1、首先下载hadoop2.5.0安装到window下，然后设置坏境变量HADOOP_HOME

2、下载hadoop-common-2.2.0-bin,下载地址如下
https://github.com/srccodes/hadoop-common-2.2.0-bin

3、找到hadoop-common-2.2.0-bin压缩包bin下面的winutils.exe文件，放到$hadoop_home\bin 目录下

4、战斗奥hadoop-common-2.2.0-bin压缩包bin下的hadoop.dll文件，
复制到C:\windows\System32，目录下。

5、最后一步，运行Hadoop代码前，要限执行下列代码，就可以正常在windows下运行Hadop测试代码
```
System.setProperty("hadoop.home.dir","D:\\JAVA\\hadoop-2.5.0\\");
```


