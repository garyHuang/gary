# 编译hadoop源码

标签（空格分隔）： hadoop


[TOC]


---
#一、准备资料
> 64位linux系统。我使用的是 CentOS
1、JDK 1.7+。注：使用1.7即可，如果是1.8则会编译失败，1.6没有试过，看网上大牛的帖子说也能过
2、maven-3.2.5。 这是apache的一个产品，hadoop的编译要就是3.0以上
3、protobuf 注：谷歌的产品，最好是提前百度准备一下这个文件
4、hadoop-2.5.2-src 这个可以到Apache的官网上去下载
5、ant-1.9.4 这个也是Apache的，在文章最后附的参考链接中有关于下载的

# 二、编译hadoop源码步骤

### 1. 下载hadoop源码
      下载地址：https://archive.apache.org/dist/hadoop/common/hadoop-2.5.0/
     图片：![图片][1]
  [1]: http://static.zybuluo.com/Great-Chinese/y40gk95afoopogkheqoo8ys4/1.jpg
  
### 2、安装maven
> 这个和安装jdk是一样的，安装成后同样需要配置环境变量
 export MAVEN_HOME=/opt/soft/modules/apache-maven-3.0.5
 export PATH=.:$PATH:$MAVEN_HOME/bin
 输入 source /etc/profile 生效
 
### 3、安装protobuf
这一步比较关键，protobuf最好是提前下载然后上传上去。安装protobuf前，需要安装一些其他的东西
protobuf-2.5.0.tar.gz 下载地址： http://download.csdn.net/detail/hfmbook/9251469
https://github.com/google/protobuf/releases 是google的开源项目
> yum  install  gcc    安装c++
  yum  install  gcc-c++          然后会两次提示输入 y（yes/no）
  yum install  make          可能会提示因为make已经是最新版本，而不会安装，这个无所谓，反正是最新版本，就不安装了.
 
  
接下来便是安装protobuf，同样是一个解压缩的过程
 tar -zxvf protobuf-2.5.0.tar.gz -C ./modeles/ 
 然后进入到安装目录中，以此输入一下命令
```
   cd  /protobuf-2.5.0
    ./configrue <!--如果提示没有权限执行，则执行 sudo chown 755 configrue-->
    make <!-- 执行时间很长 -->
    make  install 
```
上司命令执行完毕后测试 是否安装成功
> protoc  --version 如果提示此命令不存在，则从新执行上面三条命令

### 4、安装CMake
```
yum  install  cmake     
yum  install  openssl-devel
um  install  ncurses-devel
```
### 5、ant 安装   
  tar zxvf  apache-ant-1.9.4-bin.tar.gz -C ./modules/
 直接解压ant包即可
 配置ant环境变量
 ```
 export ANT_HOME=/opt/soft/modules/apache-ant-1.9.6
 export PATH=.:$PATH:$ANT_HOME/bin
 ```
 配置后 执行  source /etc/profile 生效
 检查是否成功：ant  -version

### 6、编译hadoop
 a、进入hadop操作文件夹
```
 tar -zxvf hadoop-2.5.2-src.tar.gz -C ./modules/
 cd ./modules/hadoop-2.5.2-src/
 
```
b、编译hadoop
 ```
  mvn package -Pdist,native -DskipTests -Dtar
 ```
 如果命令执行失败，则重新安装一次即可以。
 jar包默认从国外的网站下载可能很慢 建议在 maven根目录下的 ./confg/settings.xml 的mirrors节点下配置：
 
```
<mirror>
    <id>nexus-osc</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus osc</name>
    <url>http://maven.oschina.net/content/groups/public/</url>
</mirror>
      
<mirror>
    <id>mvnrepository-osc</id>
    <mirrorOf>*</mirrorOf>
    <name>mvnrepository osc</name>
    <url>http://central.maven.org/maven2/</url>
</mirror>
<!-- 国内的maven镜像 -->
<mirror>  
	<id>planetmirror.com</id>  
	<name>PlanetMirror Australia</name>  
	<url>http://124.192.56.196:8081/nexus/content/groups/public2/</url>  
	<mirrorOf>central</mirrorOf>  
</mirror> 
```
 配置后maven下载jar包就会从国内的网站 oschina网站下载