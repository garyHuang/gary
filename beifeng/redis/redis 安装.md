# redis 安装

标签（空格分隔）： redis 

[toc]

---
##1、下载
http://www.redis.io/download/
##2、解压

##3、使用 make 编译
```
make
```

##4、编译成功后，拷贝核心文件到redis安装目录

##5、让redis数据库在后台启动
修改配置文件： redis.conf
```
找到 daemonize no这行代码
修改为：daemonize yes
```
##6、启动redis数据库
```
./redis-server redis.conf
```
##7、重启redis数据库
> ./redis-cli shutdown
./redis-server redis.conf

##8、redis-cli使用
```
 redis-cli -h 127.0.0.0 -p 6379  #远程登录
 redis-cli   #默认登录本地
```
##9、操作名称
 普通字符串修改
 set foo gary
 get foo
 LRANGE 操作
 LPUSH foo 1
 LPUSH foo 2
 LRANGE foo 0 -1
##10、集群 
###10.1 部署至少6台redis服务器
我这里部署在一台机器上，设置六个端口分别如下：
```
192.168.1.21:6032
192.168.1.21:6033
192.168.1.21:6034
192.168.1.21:6035
192.168.1.21:6036
192.168.1.21:6037
192.168.1.21:6038
192.168.1.21:6039
```
###10.2检查各个redis是否启动正常
```
[root@local01 redis2]# ps -ef | grep redis
root      8506     1  0 13:50 ?        00:00:04 ./redis-server 192.168.1.21:6376 [cluster]
root      8613     1  0 13:50 ?        00:00:04 ./redis-server 192.168.1.21:6377 [cluster]
root      8618     1  0 13:50 ?        00:00:04 ./redis-server 192.168.1.21:6378 [cluster]
root      8623     1  0 13:50 ?        00:00:04 ./redis-server 192.168.1.21:6379 [cluster]
root     12437     1  0 14:41 ?        00:00:00 ./redis-server 192.168.1.21:6375 [cluster]
root     12441     1  0 14:41 ?        00:00:00 ./redis-server 192.168.1.21:6374 [cluster]
root     12445     1  0 14:41 ?        00:00:00 ./redis-server 192.168.1.21:6373 [cluster]
root     12449     1  0 14:41 ?        00:00:00 ./redis-server 192.168.1.21:6372 [cluster]
root     12887 12081  0 14:52 pts/1    00:00:00 grep --color=auto redis
```
###10.3创建集群(redis-trib.rb文件在redis源码目录的src下面，直接复制到需要使用的机器就可以了)
```
./redis-trib.rb create --replicas 1 192.168.1.21:6376 192.168.1.21:6377 192.168.1.21:6378 192.168.1.21:6379 192.168.1.21:6375 192.168.1.21:6374 192.168.1.21:6373 192.168.1.21:6372
>>> Creating cluster
Connecting to node 127.0.0.1:7000: OK
Connecting to node 127.0.0.1:7001: OK
Connecting to node 127.0.0.1:7002: OK
Connecting to node 127.0.0.1:7003: OK
Connecting to node 127.0.0.1:7004: OK
Connecting to node 127.0.0.1:7005: OK
>>> Performing hash slots allocation on 8 nodes...
。。。还有些信息没有拷贝上来
```
###10.4如果命令运行不成功，是因为本地没有安装ruby环境.
yum -y install ruby rubygems
###10.5安装redis依赖
gem install redis 安装，如果卡住不动，就收工下载文件
wget https://rubygems.global.ssl.fastly.net/gems/redis-3.2.1.gem
gem install -l ./redis-3.2.1.gem
安装完成后，在执行 10.3 的集群命令