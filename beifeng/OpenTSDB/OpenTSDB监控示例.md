# OpenTSDB监控示例

标签（空格分隔）： OpenTSDB

##监控脚本（监控tsdb自身情况）
```
$ vi loadavg-collector.sh

#!/bin/bash
INTERVAL=10
while :; 
    do echo stats || exit; 
    sleep $INTERVAL; 
done | nc -w 30 192.168.84.128 4242 | sed 's/^/put /' | nc -w 30 192.168.84.128 4242

##INTERVAL=10，时间周期10秒



##保存以上脚本，启动监控
$ sh loadavg-collector.sh &

```
<font color="red">查看图形界面时注意与Linux时间</font>
![QQ截图20160108233801.png-94.6kB][1]



---

在此输入正文


  [1]: http://static.zybuluo.com/awsekfozc/z1ghi190c5tbfk1bz8jeiosq/QQ%E6%88%AA%E5%9B%BE20160108233801.png