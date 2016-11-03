# OpenTSDB 监控报警接入

标签（空格分隔）： OpenTSDB


    参考：http://www.ttlsa.com/opentsdb/opentsdb-nagios-monitoring-and-alarming-realization/


##1）脚本说明
> 在OpenTSDB源码tools目录下有一个Python工具check_tsd。该脚本查询OpenTSDB并返回兼容Nagios的输出OK/WARNING/CRITICAL状态格式。

```python
#!/usr/bin/python
#
# 该脚本根据给定的metric查询TSDB，并与指定的阀值进行比较。
# 兼容Nagios的输出格式，因此可以作为nagios的命令。
#
# check_tsd -m mysql.slave.seconds_behind_master -t host=foo -t schema=mydb
#     -d 600 -a avg -x gt -w 50 -c 100
#
 
import httplib
import operator
import socket
import sys
import time
from optparse import OptionParser
 
def main(argv):
    """从TSDB提取数据，并做简单的Nagios报警。"""
 
    parser = OptionParser(description='Simple TSDB data extractor for Nagios.')
    parser.add_option('-H', '--host', default='localhost', metavar='HOST',
            help='Hostname to use to connect to the TSD.')
    parser.add_option('-p', '--port', type='int', default=80, metavar='PORT',
            help='Port to connect to the TSD instance on.')
    parser.add_option('-m', '--metric', metavar='METRIC',
            help='Metric to query.')
    parser.add_option('-t', '--tag', action='append', default=[],
            metavar='TAG', help='Tags to filter the metric on.')
    parser.add_option('-d', '--duration', type='int', default=600,
            metavar='SECONDS', help='How far back to look for data.')
    parser.add_option('-D', '--downsample', default='none', metavar='METHOD',
            help='Downsample function, e.g. one of avg, min, sum, or max.')
    parser.add_option('-W', '--downsample-window', type='int', default=60,
            metavar='SECONDS', help='Window size over which to downsample.')
    parser.add_option('-a', '--aggregator', default='sum', metavar='METHOD',
            help='Aggregation method: avg, min, sum (default), max.')
    parser.add_option('-x', '--method', dest='comparator', default='gt',
            metavar='METHOD', help='Comparison method: gt, ge, lt, le, eq, ne.')
    parser.add_option('-r', '--rate', default=False,
            action='store_true', help='Use rate value as comparison operand.')
    parser.add_option('-w', '--warning', type='float', metavar='THRESHOLD',
            help='Threshold for warning.  Uses the comparison method.')
    parser.add_option('-c', '--critical', type='float', metavar='THRESHOLD',
            help='Threshold for critical.  Uses the comparison method.')
    parser.add_option('-v', '--verbose', default=False,
            action='store_true', help='Be more verbose.')
    parser.add_option('-T', '--timeout', type='int', default=10,
            metavar='SECONDS',
            help='How long to wait for the response from TSD.')
    parser.add_option('-E', '--no-result-ok', default=False,
            action='store_true',
            help='Return OK when TSD query returns no result.')
    parser.add_option('-I', '--ignore-recent', default=0, type='int',
            metavar='SECONDS', help='Ignore data points that are that'
            ' are that recent.')
    parser.add_option('-S', '--ssl', default=False, action='store_true',
            help='Make queries to OpenTSDB via SSL (https)')
    (options, args) = parser.parse_args(args=argv[1:])
 
    # 验证参数
    if options.comparator not in ('gt', 'ge', 'lt', 'le', 'eq', 'ne'):
        parser.error("Comparator '%s' not valid." % options.comparator)
    elif options.downsample not in ('none', 'avg', 'min', 'sum', 'max'):
        parser.error("Downsample '%s' not valid." % options.downsample)
    elif options.aggregator not in ('avg', 'min', 'sum', 'max'):
        parser.error("Aggregator '%s' not valid." % options.aggregator)
    elif not options.metric:
        parser.error('You must specify a metric (option -m).')
    elif options.duration <= 0:
        parser.error('Duration must be strictly positive.')
    elif options.downsample_window <= 0:
        parser.error('Downsample window must be strictly positive.')
    elif options.critical is None and options.warning is None:
        parser.error('You must specify at least a warning threshold (-w) or a'
                     ' critical threshold (-c).')
    elif options.ignore_recent < 0:
        parser.error('--ignore-recent must be positive.')
 
    if not options.critical:
        options.critical = options.warning
    elif not options.warning:
        options.warning = options.critical
 
    # 处理标签
    tags = ','.join(options.tag)
    if tags:
        tags = '{' + tags + '}'
 
    # 组装URL并获取
    if options.downsample == 'none':
        downsampling = ''
    else:
        downsampling = '%ds-%s:' % (options.downsample_window,
                                    options.downsample)
    if options.rate:
        rate = 'rate:'
    else:
        rate = ''
    url = ('/q?start=%ss-ago&m=%s:%s%s%s%s&ascii&nagios'
           % (options.duration, options.aggregator, downsampling, rate,
              options.metric, tags))
    tsd = '%s:%d' % (options.host, options.port)
    if options.ssl:  # Pick the class to instantiate first.
      conn = httplib.HTTPSConnection
    else:
      conn = httplib.HTTPConnection
    if sys.version_info[0] * 10 + sys.version_info[1] >= 26:  # Python >2.6
      conn = conn(tsd, timeout=options.timeout)
    else:  # Python 2.5 or less, using the timeout kwarg will make it croak :(
      conn = conn(tsd)
    try:
      conn.connect()
    except socket.error, e:
      print "ERROR: couldn't connect to %s: %s" % (tsd, e)
      return 2
    if options.verbose:
        peer = conn.sock.getpeername()
        print 'Connected to %s:%d' % (peer[0], peer[1])
        conn.set_debuglevel(1)
    now = int(time.time())
    try:
      conn.request('GET', url)
      res = conn.getresponse()
      datapoints = res.read()
      conn.close()
    except socket.error, e:
      print "ERROR: couldn't GET %s from %s: %s" % (url, tsd, e)
      return 2
 
    # URL请求失败时
    if res.status not in (200, 202):
        print ('CRITICAL: status = %d when talking to %s:%d'
               % (res.status, options.host, options.port))
        if options.verbose:
            print 'TSD said:'
            print datapoints
        return 2
 
    # URL请求成功时
    if options.verbose:
        print datapoints
    datapoints = datapoints.splitlines()
 
    def no_data_point():
 """从TSDB没有获取到任何数据时"""
        if options.no_result_ok:
            print 'OK: query did not return any data point (--no-result-ok)'
            return 0
        else:
            print 'CRITICAL: query did not return any data point'
            return 2
 
    if not len(datapoints):
        return no_data_point()
 
    comparator = operator.__dict__[options.comparator]
    rv = 0         # 该脚本返回值，0-OK,1-WARNING,2-CRITICAL
    badts = None   # 超过阀值的时间戳
    badval = None  # 超过阀值的值
    npoints = 0    # 查询到多少数据点？
    nbad = 0       # 有多少数据点超过阀值？
    for datapoint in datapoints:
        datapoint = datapoint.split()
        ts = int(datapoint[1])
        delta = now - ts
        if delta > options.duration or delta <= options.ignore_recent:
            continue  # 忽略不在options.duration或options.ignore_recent的数据点
        npoints += 1
        val = datapoint[2]
        if '.' in val:
            val = float(val)
        else:
            val = int(val)
        bad = False  # Is the current value bad?
        # 比较 warning/crit
        if comparator(val, options.critical):
            rv = 2
            bad = True
            nbad += 1
        elif rv < 2 and comparator(val, options.warning):
            rv = 1
            bad = True
            nbad += 1
        if (bad and
            (badval is None  # First bad value we find.
             or comparator(val, badval))):  # Worse value.
            badval = val
            badts = ts
 
    if options.verbose and len(datapoints) != npoints:
        print ('ignored %d/%d data points for being more than %ds old'
               % (len(datapoints) - npoints, len(datapoints), options.duration))  #忽略量/总查询量 数据点大于duration时间
    if not npoints:
        return no_data_point()
    if badts:
        if options.verbose:
            print 'worse data point value=%s at ts=%s' % (badval, badts)  # 糟糕数据点的值以及该点值的时间戳
        badts = time.asctime(time.localtime(badts))
 
    # 在NRPE里，字符串'|'有特殊含义，但在tag搜索时有用到。对其替换。
    ttags = tags.replace("|",":")
    if not rv:
        print ('OK: %s%s: %d values OK, last=%r'
               % (options.metric, ttags, npoints, val))  # OK: metric{tags}: 数据点数量 values OK, 最后一次的值
    else:
        if rv == 1:
            level = 'WARNING'
            threshold = options.warning
        elif rv == 2:
            level = 'CRITICAL'
            threshold = options.critical
        print ('%s: %s%s %s %s: %d/%d bad values (%.1f%%) worst: %r @ %s'  
               % (level, options.metric, ttags, options.comparator, threshold,
                  nbad, npoints, nbad * 100.0 / npoints, badval, badts))
    return rv
 
 
if __name__ == '__main__':
    sys.exit(main(sys.argv))
```

##2）参数说明

```
Options:
  -h, --help            打印帮助信息
  -H HOST, --host=HOST  指定TSDB服务器
  -p PORT, --port=PORT  指定TSDB端口
  -m METRIC, --metric=METRIC
                        查询的metric
  -t TAG, --tag=TAG     metric过滤的tags
  -d SECONDS, --duration=SECONDS
                        查询多久的数据，默认600s
  -D METHOD, --downsample=METHOD
                        缩减像素采样函数有avg, min, sum, or max.
  -W SECONDS, --downsample-window=SECONDS
                        缩减像素采样窗口大小
  -a METHOD, --aggregator=METHOD
                        聚合方法: avg, min, sum (default), max.
  -x METHOD, --method=METHOD
                        比较法: gt, ge, lt, le, eq, ne.
  -r, --rate            使用速率值作为比较的操作数
  -w THRESHOLD, --warning=THRESHOLD
                        warning阀值.
  -c THRESHOLD, --critical=THRESHOLD
                        critical阀值.
  -v, --verbose         显示详细信息
  -T SECONDS, --timeout=SECONDS
                        等待TSDB响应时间
  -E, --no-result-ok    当TSD查询没有结果返回时，返回OK状态
  -I SECONDS, --ignore-recent=SECONDS
                        忽略最近的数据点
  -S, --ssl             通过SSL查询OpenTSDB (https)
```

##3）报警接入

[Nagios服务器搭建][1]

    将check_tsd脚本放在Nagios服务器/usr/local/nagios/libexec目录里。
    编辑/usr/local/nagios/etc/objects/commands.cnf文件，添加下面的命令：
```
define command{
 command_name check_tsd
 command_line $USER1$/check_tsd -H 10.0.101.145 -p 4242 -t host=$HOSTADDRESS$ $ARG1$
}

###添加监控项
define service {
    use                         MongoDB
    host_name                   10.0.0.166
    service_description         query opcounters per second
    check_command               check_tsd!-m rate:mongo.opcounters -t type=query -d 60 -w 300 -c 500
}
```



---

在此输入正文


  [1]: http://www.2cto.com/os/201402/278528.html