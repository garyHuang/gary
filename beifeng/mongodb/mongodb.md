# mongodb

标签（空格分隔）： mongodb

---
##1、安装mongodb 
#####a、下载地址
http://www.mongodb.org/downloads
#####b、安装mongodb
#####c、启动mongodb
> mongod --dbpath "D:/JAVA/mongodb/data/" 
bin/mongod --dbpath "/hksdata/mongodb/data/" 
//其中 D:/JAVA/mongodb/data/ 是mongodb的数据目录，一定要提前创建好

#####d、window mongodb 后台启动
> mongod --dbpath "D:/JAVA/mongodb/data/" -logpath "D:/JAVA/mongodb/logs/log" -auth -service
// 其中 D:/JAVA/mongodb/logs/是目录，log是文件
bin/mongod --dbpath "/hksdata/mongodb/data/" --logpath "/hksdata/mongodb/logs/mogond.log"

##2、命令窗口链接mongodb
直接运行命令 mongodb 可默认链接本机的mongodb服务器
#####a、常用命令
> db.test.save( {"a":1 , "b":3}) //添加一条数据到test数据库
db.test.find() //test集合中所有数据
show dbs //显示当前所有的数据库
use test //切换当前数据库
db.joke.find().limit ( 2 ).skip( 100 ) ; // joke 集合名称，limit查询多少条，skip从多条开始
db.parts.find({"brandcode" : "131"});
db.parts.find({"materialName" : {$regex : '起动机' , $options : 'i' }}); //模糊搜索
db.parts.ensureIndex({"materialcode":1},{"unique":true})//指定配件编号为唯一标识
db.parts.ensureIndex({"materialcode":1 ，， ,"state":1},{"unique":true})//指定联合索引唯一标识

#####b、设置账号访问
```
//创建超级管理账号
db.createUser({
user:"admin" , 
pwd : "garyisadmin" ,
roles:[
"dbAdminAnyDatabase" 
]
})
//创建数据库管理账号
db.createUser({user:'hks',pwd:'hks123',roles:[{role:'dbOwner',db:'hks'}]}) 
//授权角色
db.grantRolesToUser("hks",[{ role: "dbOwner", db: "hks" } ]) 
```

###3linux 启动mongodb
 a、新建文件 mongodb.conf
```
dbpath=/hksdata/mongodb/data/ #数据文件目录
logpath=/hksdata/mongodb/logs/mogondb.log #日志文件
logappend=true #日志自动合并
fork=true #守护进程
bind_ip=115.28.184.227,10.45.19.14,127.0.0.1 #启动绑定可以访问的ip地址
auth=true #启动验证

```
b、启动mogodb
--fork 为后台启动
```
bin/mongod -f mongodb.conf --fork
```


