# hadoop集群--zookeeper集群

标签（空格分隔）： hadoop集群


[toc]


---
要求：zookper集群。
简介：ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。
ZooKeeper包含一个简单的原语集，提供Java和C的接口。

## zookper下载地址
    http://zookeeper.apache.org/

##配置linux无密钥登录
 配置方式：https://www.zybuluo.com/Great-Chinese/note/221102
 
##我将zookeeper文件上传到 /opt/app/ 目录下
 然后使用解压到/opt/app/ 目录下
>  tar -zxvf zookeeper-3.4.5.tar.gz -C /opt/app/

##修改 zoo_sample.cfg 为 zoo.cfg，在这个文件中加入配置
```
# 时常基准，2000毫秒
tickTime=2000
##心跳时间，20秒钟无响应 就等于改机器已经挂掉了
initLimit=10
## 同步时间为10秒
syncLimit=5
#zookeeper端口号
clientPort=2181
#zookeeper数据文件位置
dataDir=/opt/app/zookeeper-3.4.5/data
server.1=hadoop00:2888:3888
server.2=hadoop01:2888:3888
server.3=hadoop02:2888:3888
```
#####配置解说：
 server.X 代表zookeeper的机器个数，hadoop00代表你主机名称，
 dataDir表示数据文件存放的目录。

##配置myid
在我们配置的dataDir指定的目录下面，创建一个myid文件，里面内容为一个数字，用来标识当前主机，conf/zoo.cfg文件中配置的server.X中X为什么数字，则myid文件中就输入这个数字。

##启动zookeeper
 分别进入某台机器，到zookeeper的安装目录下，输入命令
 bin/zkServer.sh start
 启动zookeeper。

##查看启动状态
 ![zookeeper01.png-6.5kB][1]
 follower表示子节点
 ![zookeeper02.png-7.9kB][2]
 leader表主节点
 其他状态均为启动失败
 
 进入hadoophome目录格式化集群zookeeper
 > bin/hdfs zkfc -formatZK 
 
##客户端测试
```
    bin/zkCli.sh -server hadoop00:2181 
```
成功提示：
![zookeeper04.png-87.5kB][3]
输入一些zookeeper常用命令,进行测试操作
```
  ls /
  create /zk mydata
  delete /zk
```

  [1]: http://static.zybuluo.com/Great-Chinese/a6e4v54d1i2n1v6jq5hoyos0/zookeeper01.png
  [2]: http://static.zybuluo.com/Great-Chinese/r3omrgm0ho66i8v314hyjtff/zookeeper02.png
  [3]: http://static.zybuluo.com/Great-Chinese/e4ga6grcac5awal4o72gkd2g/zookeeper04.png