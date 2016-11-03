# OpenTSDB安装

标签（空格分隔）： OpenTSDB

##
##依赖
    A Linux system
    Java Development Kit 1.6 or later
    GnuPlot 4.2 or later
    Autotools
    Make
    Python
    Git
    An Internet connection
    
##安装（版本2.1.1）

###下载
[下载地址1][1]
[网盘地址][2]  访问密码 3db8

###安装Hbase
[Hbase安装][3]，安装好后启动Hbase

###安装Gnuplot
```
$ yum install gnuplot
```

###安装OpenTSDB

- 解压
```
$ tar -zxvf opentsdb-2.1.1.tar.gz -C /opt/moduels/
```
- build
```
$ cd /opt/moduels/opentsdb-2.1.1/
$ ./build.sh
```
- 在Hbase上创建OpenTSDB默认表
```
$ env COMPRESSION=NONE HBASE_HOME=/opt/moduels/hbase-0.98.6-hadoop2 ./src/create_table.sh
```
![QQ截图20160108212951.png-35.4kB][4]

- 配置
```
###文件位置  ./src/opentsdb.conf
#端口
tsd.network.port = 4242
tsd.http.staticroot = build/staticroot
#缓存目录
tsd.http.cachedir = /opt/moduels/opentsdb-2.1.1/data
#是否允许自动创建指标
tsd.core.auto_create_metrics = true

```
- 设置定时任务
```
## tsd.http.cachedir目录会缓存大量的临时过期文件，需要定期清理。源码目录下有提供一个脚本来定期清理该目录。可以设置为cron任务。

# crontab -e
30 * * * * /opt/moduels/opentsdb-2.1.1/tools/clean_cache.sh
```

- 启动tsdb
```
#第一种
$ nohup ./build/tsdb tsd --config=./src/opentsdb.conf 2>&1 > /dev/null  &
#第二种
$ nohup ./build/tsdb tsd --port=4242 --staticroot=build/staticroot --cachedir="/opt/moduels/opentsdb-2.1.1/data" --zkquorum=myhost:2181 &
```
- http://hadoop.zc.com:4242/
![QQ截图20160108213312.png-36.7kB][5]


---

在此输入正文


  [1]: https://github.com/OpenTSDB/opentsdb/releases
  [2]: http://yunpan.cn/cubCjy3JACNqw
  [3]: https://www.zybuluo.com/awsekfozc/note/259421
  [4]: http://static.zybuluo.com/awsekfozc/qkvkivc9hew2ixoj4v4mkksk/QQ%E6%88%AA%E5%9B%BE20160108212951.png
  [5]: http://static.zybuluo.com/awsekfozc/spexof7pralkmqy6o5gxtwa4/QQ%E6%88%AA%E5%9B%BE20160108213312.png