# OpenTSDB TCollector 详解

标签（空格分隔）： OpenTSDB

    参考：http://www.ttlsa.com/opentsdb/opentsdb-tcollector-detail/

##1）简介
    tcollector是一个客户端程序，用来收集本机的数据，并将数据发送到OpenTSDB。
    OpenTSDB被设计的收集和写入数据非常简单，有一个简单的协议，即使是一个shell脚本也可以用来发送数据。
    但是，做到可靠和一致性就有些困难了。当TSD服务器down了该怎么做？如何确保采集者一直在运行？这就是要使用tcollector的原因了。
    tcollector可以为你做一下几件事：

     - 运行所有的采集者并收集数据
     - 完成所有发送数据到TSD的连接管理任务
     - 不必在你写的每个采集者中嵌入这些代码
     - 是否删除重复数据
     - 处理所有有线协议，以后今后的改进

- 重复数据删除

>    通常情况下，你需要收集你系统的所有数据。这会产生大量的数据点，其中大多数的数据是不经常改变的。但是，当它们改变时，你想以细粒度来解决。Tcollector会记住发送所有时间序的最后的值和时间戳，如果该值在采样间隔内没有发生变化，将抑制发送该数据点。一旦该值发生变化(或10分钟后)，则发送上次抑制的值和时间戳，以及当前的值和时间戳。这样一来，所有的图形将是正确的。
    重复数据删除技术会减少TSD数据点的数量，同样也可以减少网络负载。
    未来版本的OpenTSDB，会使用RLE来改善存储格式，使得很少存储重复值。
    
- 使用tcollector收集大量的metric

> 在tcollector下，可以使用任何语言编写采集者。只需要采集者有可执行权限，并把数据以标准输出输出。其他的交给tcollector处理。采集者位于采集器目录下，tcollector会遍历每个数字目录并执行这些目录下的采集者。数字目录代表采集间隔。如，如果目录名称为60，tcollector将尝试每60s运行该目录下的所有采集者。目录名为0，说明采集者持续长久运行。tcollector会读取它们的输出，如果它们挂掉了，会复活它们。
一般来说，长久运行的采集者需要考虑耗资料少。OpenTSDB被设计成每个metric具有大量数据点。对于大多数metric，通常以15s为一个数据点。
如果以任何非数字命令目录，将忽略该目录下的所有采集者。采集器目录下还包含lib和etc目录，为所有采集者使用的library和配置数据。

##2）安装tcollector

- [下载][1]，下载后解压到${OPENTSDB_HOME}目录下
 
- 修改
```
修改 ${OPENTSDB_HOME}/tcollector-master/startstop文件
##修改下面这个变量，tsd的host
TSD_HOST=hadoop.zc.com
```


##2）tcollector自带的监控脚本
###0/dfstat.py
这些统计类似于/usr/bin/df工具提供的。
df.bytes.total  数据总大小
df.bytes.used  已用的字节数
df.bytes.free    剩余的字节数
df.inodes.total  总的inode数量
df.inodes.used  已用的inode数
df.inodes.free  剩余的inode数
这些metric包含时间序标记每个挂载点和文件系统类型。此采集器可通过cgroup, debugfs, devtmpfs, rpc_pipefs, rootfs filesystems和挂载点/dev/, /sys/, /proc/和/lib/进行过滤。

###0/ifstat.py
这些统计来自/proc/net/dev。
proc.net.bytes    (rate) Bytes in/out
proc.net.packets  (rate) Packets in/out
proc.net.errs  (rate) Packet errors in/out
proc.net.dropped (rate) Dropped packets  in/out
接口标签iface=, 方向标签direction=in|out。 仅仅ethN接口采集，有意排除bondN接口，因为bonded接口也就是各个ethN接口的总计，没必要重复收集。

###0/iostat.py
数据来源于/proc/diskstats.
iostat.disk.*  每个磁盘的统计
iostat.part.* 每个分区的统计
iostats内容参见：https://www.kernel.org/doc/Documentation/iostats.txt

###0/netstat.py
socket分配和网龙统计信息。
从/proc/net/sockstat来的metric。
net.sockstat.num_sockets  sockets分配的数量，仅TCP
net.sockstat.num_timewait   TCP sockets当前处在TIME_WAIT状态数量
net.sockstat.sockets_inuse 套接字使用的数量
net.sockstat.num_orphans 孤儿套接字数量(不依附于任何文件描述符)
net.sockstat.memory 分配给该套接字类型的内存大小（字节）
net.sockstat.ipfragqueues 等待重新组装的IP流数量
从 /proc/net/netstat (netstat -s )来的metric。
net.stat.tcp.abort  内核中止连接数量
net.stat.tcp.abort.failed 内核中止失败数量，因为没有足够内存来复位。
net.stat.tcp.congestion.recovery 内存检测到假重传，能部分回收或全部CWND数量。
net.stat.tcp.delayedack 发送不同类型的延迟确认数量。
net.stat.tcp.failed_accept 在3WHS连接之后丢弃数。
net.stat.tcp.invalid_sack 无效的SACK数量。
net.stat.tcp.memory.pressure  进入"memory pressure"的次数。
net.stat.tcp.memory.prune  因内存不足放弃接收数据的次数。
net.stat.tcp.packetloss.recovery  丢失恢复的次数。
net.stat.tcp.receive.queue.full 因套接字接收队列慢导致被丢弃接收到的数据包数量。
net.stat.tcp.reording  检测到重新排序的次数。
net.stat.tcp.syncookies SYN cookies数。

###0/nfsstat.py
这些统计来自/proc/net/rpc/nfs.
nfs.client.rpc.stats  RPC状态统计
nfs.client.rpc  RPC调用统计

###0/procnettcp.py
这些统计来自 /proc/net/tcp{,6}。每60s收集一次。
 proc.net.tcp  TCP连接数

###0/procstats.py
来自/proc的统计。
proc.stat.cpu  CPU使用率统计。标签为CPU类型。type=user, nice, system, idle, iowait, irq, softirq。
proc.stat.intr  中断率
proc.stat.ctxt 上下文切换率
procstat内容参见：http://www.linuxhowtos.org/System/procstat.htm
proc.vmstat.*  从/proc/vmstat信息，参见：http://www.linuxinsight.com/proc_vmstat.html
proc.meminfo.* 从/proc/meminfo统计的内存使用情况
proc.loadavg.*  从/proc/loadavg统计的1min, 5min, 15min, runnable, total_threads指标。
proc.uptime.total 启动率
proc.uptime.now
proc.kernel.entropy_avail
sys.numa.zoneallocs
sys.numa.foreign_allocs
sys.numa.allocation
sys.numa.interleave

###0/smart-stats.py
统计SMART磁盘信息。
smart.raw_read_error_rate  当从盘面上读取数据时，有关硬件读取数据出错的几率。
smart.throughput_performance  硬盘驱动的总体吞吐量
smart.spin_up_time  主轴旋转起来的平均时间（从零转速到完全运行[毫秒]）
smart.start_stop_count  主轴开始/停止周期的统计
smart.reallocated_sector_ct  重新分配扇区的统计
smart.seek_error_rate 磁头寻道错误几率
smart.seek_time_performance 磁头寻道平均时间
smart.power_on_hours
smart.spin_retry_count
smart.recalibration_retries
smart.power_cycle_count
smart.soft_read_error_rate
smart.program_fail_count_chip
smart.erase_fail_count_chip
smart.wear_leveling_count
smart.used_rsvd_blk_cnt_chip
smart.used_rsvd_blk_cnt_tot
smart.unused_rsvd_blk_cnt_tot
smart.program_fail_cnt_total
smart.erase_fail_count_total
smart.runtime_bad_block
smart.end_to_end_error
smart.reported_uncorrect
smart.command_timeout
smart.high_fly_writes
smart.airflow_temperature_celsius
smart.g_sense_error_rate
smart.power-off_retract_count
smart.load_cycle_count
smart.temperature_celsius
smart.hardware_ecc_recovered
smart.reallocated_event_count
smart.current_pending_sector
smart.offline_uncorrectable
smart.udma_crc_error_count
smart.write_error_rate
smart.media_wearout_indicator
smart.transfer_error_rate
smart.total_lba_writes
smart.total_lba_read
SMART说明参见：https://en.wikipedia.org/wiki/S.M.A.R.T.#Known_ATA_S.M.A.R.T._attributes
理解这些metric最好是看厂家的说明。
其他采集器

###0/couchbase.py
统计couchbase数据库的。
所有的metric都以bucket=为标签。桶是Couchbase服务器集群中的逻辑分组。参见：http://docs.couchbase.com/couchbase-manual-2.1/#cbstats-tool

###0/elasticsearch.py
统计Elastic 搜索的。
metric说明参见http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/cluster.html

###0/hadoop_datanode_jmx.py
统计Hadoop  DataNode 状态。
默认情况下，采集器对这些metric是禁用的。revision, hdfsUser, hdfsDate, hdfsUrl, date, hdfsRevision, user, hdfsVersion, url, version, NamenodeAddress, Version, RpcPort, HttpPort, CurrentThreadCpuTime, CurrentThreadUserTime, StorageInfo, VolumeInfo.
metric说明参见：http://hbase.apache.org/book.html#hbase_metrics

###0/haproxy.py
统计Haproxy 状态信息。
haproxy.current_sessions 当前的session数
haproxy.session_rate 每秒新增session量
所有metric的标签有server (server=) 和cluster (cluster=).
具体信息说明参见：http://haproxy.1wt.eu/download/1.4/doc/configuration.txt

###0/hbase_regionserver_jmx.py
统计Hadoop  RegionServer 信息
默认情况下，采集器对这些metric是禁用的。revision, hdfsUser, hdfsDate, hdfsUrl, date, hdfsRevision, user, hdfsVersion, url, version, Version, RpcPort, HttpPort,HeapMemoryUsage, NonHeapMemoryUsage.
具体说明参见：http://hbase.apache.org/book.html#hbase_metrics

###0/mongo.py
统计Mongo 信息。
具体metric说明参见：http://docs.mongodb.org/manual/reference/server-status/

###0/mysql.py
统计mysql信息。
统计的信息有：InnoDB Innodb monitors, Global Show status, Engine Show engine, Slave Show slave status, Process list Show process list.

###0/postgresql.py
统计PostgreSQL 信息。
metric说明参见：http://www.postgresql.org/docs/9.2/static/monitoring-stats.html

###0/redis-stats.py
统计redis信息。
metric说明参见：http://redis.io/commands/INFO

###0/riak.py
统计riak信息。
metric说明参见：http://docs.basho.com/riak/latest/ops/running/stats-and-monitoring/#Statistics-from-Riak

###0/varnishstat.py
统计varnish信息。
默认情况下，所有的metric都收集。若要改变编辑采集器的vstats数组。运行“varnishstat-l”列出所有可用的metric。

###0/zookeeper.py
统计zookeeper信息。



---

在此输入正文


  [1]: https://github.com/OpenTSDB/tcollector