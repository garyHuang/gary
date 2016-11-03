# 编译 snappy

标签（空格分隔）： hadoop ubuntu

[TOC]

---
###1、下载 snappy
下载 https://github.com/google/snappy 
a、进入解压后的目录，使用命令：
```
./configure
make
make install
```
b、下载 hadoop-snappy
    下载地址：https://github.com/garyHuang/snappy
需要安装的软件：
    gcc c++, autoconf, automake, libtool, Java 6, JAVA_HOME set, Maven 3
进入加压后的文件夹：
```
mvn package -Dsnappy.prefix=/usr/local/lib
```
说明： /usr/local/lib是snappy默认安装目录
c、把编译成功的 hadoop-snappy-0.0.1-SNAPSHOT.jar 文件夹拷贝到 hadoop_home/lib目录下，把hadoop-snappy-0.0.1-SNAPSHOT.tar.gz解压，
将文件夹，hadoop-snappy-0.0.1-SNAPSHOT/lib/native/Linux-amd64-64/下的文件拷贝到 hadoop_home/lib/native 目录下。

###2、bin/hadoop checknative
看到如下图，说明 hadoop-snappy安装成功了
![001.png-19.4kB][1]


  [1]: http://static.zybuluo.com/Great-Chinese/nmdentdr2pq6oxgroj0rdojr/001.png