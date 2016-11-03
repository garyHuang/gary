# hue 大数据可视化工具

标签（空格分隔）： hue

[TOC]

---
#1)下载
下载地址
http://archive.cloudera.com/cdh5/cdh/5/
http://archive.cloudera.com/cdh5/cdh/5/hue-3.7.0-cdh5.3.6-src.tar.gz
帮助文档
https://github.com/cloudera/hue/
#2）安装编译
```
 1、将hue-3.7.0-cdh5.3.6-src.tar.gz 解压到 /opt/soft/cdh5/
 这个目录已经安装了hadoop和hive
 2、安装相关编译软件
 sudo yum install ant asciidoc cyrus-sasl-devel cyrus-sasl-gssapi gcc gcc-c++ krb5-devel libtidy libxml2-devel libxslt-devel openldap-devel python-devel sqlite-devel openssl-devel mysql-devel gmp-devel
 
3、make编译hue
make apps
修改配置文件 /opt/cdh/hue-3.7.0-cdh5.3.6/desktop/conf/hue.ini
```
#3）集成HDFS
 
```
<!-- hdfs-site.xml -->
<property>
  <name>dfs.webhdfs.enabled</name>
  <value>true</value>
</property>
<!-- core-site.xml -->
<property>
  <name>httpfs.proxyuser.hue.hosts</name>
  <value>*</value>
</property>
<property>
  <name>httpfs.proxyuser.hue.groups</name>
  <value>*</value>
</property>
<!-- httpfs-site.xml -->
<property>
  <name>httpfs.proxyuser.hue.hosts</name>
  <value>*</value>
</property>
<property>
  <name>httpfs.proxyuser.hue.groups</name>
  <value>*</value>
</property>
```

#4）配置desktop/conf/hue.ini
##4.1基本配置
```
  # Set this to a random string, the longer the better.
  # This is used for secure hashing in the session store.
  secret_key=fdsafjsdafjskl@#$@#238u234902

  # Webserver listens on this address and port
  http_host=cdh5.hadoop.com
  http_port=8888

  # Time zone name
  time_zone=Asia/Shanghai

  # Enable or disable Django debug mode.
  django_debug_mode=false

  # Enable or disable backtrace for server error
  http_500_debug_mode=false
```
启动 build/env//bin/supervisor
访问http://ip:8888/设置hue超级管理员密码，不能忘记否则 讲无法再次使用超过管理员账号
##配置数据库
在databases下配置
```
    [[databases]]
# sqlite configuration.
    [[[sqlite]]]
      # Name to show in the UI.
      nice_name=SQLite

      # For SQLite, name defines the path to the database.
      name=/opt/soft/cdh5/hue-3.7.0-cdh5.3.6/desktop/desktop.db

      # Database backend to use.
      engine=sqlite

      # Database options to send to the server when connecting.
      # https://docs.djangoproject.com/en/1.4/ref/databases/
      ## options={}

    # mysql, oracle, or postgresql configuration.
    [[[mysql]]]
      # Name to show in the UI.
      nice_name=mySqlDB
      # For MySQL and PostgreSQL, name is the name of the database.
      # For Oracle, Name is instance of the Oracle server. For express edition
      # this is 'xe' by default.
      name=test
      # Database backend to use. This can be:
      # 1. mysql
      # 2. postgresql
      # 3. oracle
      engine=mysql
      # IP or hostname of the database to connect to.
      host=cdh5.hadoop.com
      # Port the database server is listening to. Defaults are:
      # 1. MySQL: 3306
      # 2. PostgreSQL: 5432
      # 3. Oracle Express Edition: 1521
      port=3306
      # Username to authenticate with when connecting to the database.
      user=root
      # Password matching the username to authenticate with when
      # connecting to the database.
      password=123456
      # Database options to send to the server when connecting.
      # https://docs.djangoproject.com/en/1.4/ref/databases/
      ## options={}
```
##4.2配置hdfs
1） 配置 hdfs-site.xml
```
<property>
  <name>dfs.webhdfs.enabled</name>
  <value>true</value>
</property>
```
2)配置core-site.xml
```
<property>
  <name>hadoop.proxyuser.hue.hosts</name>
  <value>*</value>
</property>
<property>
  <name>hadoop.proxyuser.hue.groups</name>
  <value>*</value>
</property>
```
3)配置httpfs-site.xml
```
<property>
  <name>httpfs.proxyuser.hue.hosts</name>
  <value>*</value>
</property>
<property>
  <name>httpfs.proxyuser.hue.groups</name>
  <value>*</value>
</property>
```
4）配置hue.ini
```
[[hdfs_clusters]]
    # HA support by using HttpFs

    [[[default]]]
      # Enter the filesystem uri
      fs_defaultfs=hdfs://cdh5.hadoop.com:8020

      # NameNode logical name.
      ## logical_name=

      # Use WebHdfs/HttpFs as the communication mechanism.
      # Domain should be the NameNode or HttpFs host.
      # Default port is 14000 for HttpFs.
      webhdfs_url=http://cdh5.hadoop.com:50070/webhdfs/v1

      # Change this if your HDFS cluster is Kerberos-secured
      ##security_enabled=false

      # Default umask for file and directory creation, specified in an octal value.
      ##umask=022
	  #Directory of the Hadoop Home configuration
	  hadoop_hdfs_home=/opt/soft/cdh5/hadoop-2.5.0-cdh5.3.6
	  hadoop_bin=/opt/soft/cdh5/hadoop-2.5.0-cdh5.3.6/bin
      # Directory of the Hadoop configuration
      hadoop_conf_dir=/opt/soft/cdh5/hadoop-2.5.0-cdh5.3.6/etc/hadoop
```
##4.2配置yarn
```
[[yarn_clusters]]

    [[[default]]]
      # Enter the host on which you are running the ResourceManager
      resourcemanager_host=cdh5.hadoop.com

      # The port where the ResourceManager IPC listens on
      resourcemanager_port=8032

      # Whether to submit jobs to this cluster
      submit_to=True

      # Resource Manager logical name (required for HA)
      ## logical_name=

      # Change this if your YARN cluster is Kerberos-secured
      ##security_enabled=false

      # URL of the ResourceManager API
      resourcemanager_api_url=http://cdh5.hadoop.com:8088
		
      # URL of the ProxyServer API
      proxy_api_url=http://cdh5.hadoop.com:8088

      # URL of the HistoryServer API
      history_server_api_url=http://cdh5.hadoop.com:19888

      # In secure mode (HTTPS), if SSL certificates from Resource Manager's
      # Rest Server have to be verified against certificate authority
      ##ssl_cert_ca_verify=False

    # HA support by specifying multiple clusters
    # e.g.
```

#5）集成oozie
##5.1修改hue.ini
找到[liboozie]
```
[liboozie]
  # The URL where the Oozie service runs on. This is required in order for
  #设置oozie访问url
  oozie_url=http://cdh5.hadoop.com:11000/oozie/

  # Requires FQDN in oozie_url if enabled
  ## security_enabled=false

  # 设置oozie 远程代码位置
  remote_deployement_dir=/user/gary/gywf

###########################################################################
# Settings to configure the Oozie app
###########################################################################
[oozie]
  # 设置oozie本地代码库
  local_data_dir=/opt/soft/cdh5/oozie-4.0.0-cdh5.3.6/gywf

  #设置oozie测试代码库
  sample_data_dir=/opt/soft/cdh5/oozie-4.0.0-cdh5.3.6/examples

  # 设置oozie本地代码在hdfs上的目录
  remote_data_dir=/user/gary/gywf

  #最大运行的workflowshulle
  oozie_jobs_count=20000

  # Use Cron format for defining the frequency of a Coordinator instead of the old frequency number/unit.
  enable_cron_scheduling=true

```
##5.2修改oozie lib目录在hdfs上的位置，目录默认在/user/oozie/share/lib要修改成自己使用的目录 /user/gary/share/lib。
修改文件desktop/libs/liboozie/src/liboozie/conf.py 78行代码
```
ConfigMock('/user/gary/share/lib')
```
,我们采用cdh5同意