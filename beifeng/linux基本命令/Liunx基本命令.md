# Liunx基本命令 

标签： linux基本命令

[toc]

---

一、liunx 入门基本命令
## 1、liunx查询ip地址
    ifconfig
## 2、linux启动登录
 
 首次登录方式 账号root 密码123456
## 3、创建用户分组
--
        groupadd hadoop
## 4、创建用户
  Ubuntu系统新建用户
  useradd gary -m -s /bin/bash  
  cenos useradd gary
   创建用户后linux会默认创建用户主文件夹目录： /home/gary
     其中gary是当前用户名称。
   如果想自己指定账号的主目录，则使用命令： 
     useradd -d /home/文件夹名 gary
       
## 5、设置初始密码 
 
   passwd gary， 初始密码可以设置简单密码
## 6、修改密码
 
  passwd   直接修改当前用户的密码
  passwd gary  修改指定用户的密码（root账号使用）
##7、切换当前登录用户
 
    su gary 切换切换为gary账号（如果是root则不需要输入密码）
    su 切换为root账号，需要输入root账号密码
## 8、普通账户加入sudo权限。方法：切换到**root**用户
 
    # vi /etc/sudoers
	在第一行添加如下内容：
	gary ALL=(root)NOPASSWD:ALL
# 二、系统维护

## 1、查询linux主机名称

    hostname
## 2、设置linux主机名称
```
    hostname hadoop.gary.com
    修改hosts文件
    vi /etc/hosts
    加入 一行代码：例如本机ip为 192.168.31.176
    192.168.31.176 hadoop.gary.com hadoop.gary.com
    永久修改hostname 修改文件/etc/sysconfig/network                       中hostname修改为hadoop.gary.com
```
    
## 3、关机系统
-------------------------------------
      shutdown -r 0  立即关机
       
## 4、重启系统
 
      root 立即重启系统
## 5、查看硬盘使用情况

 df -lh 
## 6、查看某个目录占用空间

  du -sh /home/gary/
## 7、修复磁盘
> fsck /dev/sda3

## 8、挂载磁盘
> mount /dev/sdb1 /data01
     其中 /dev/sdb1 是磁盘
       /data01 是挂载目录
     
## 9、卸载磁盘

  umount /dev/sdb1
## 10、看系统内存使用情况
    free -m
##  11、查看进程情况
 ps -ef 
## 12、搜索java进程
 ps -ef | grep java
##13、压缩
tar  -zcvf ROOT.tar.gz ROOT/

##14)挂载硬盘提示
Disk /dev/vdb doesn't contain a valid partition table
![linux挂载硬盘.png-25.5kB][1]

格式化硬盘为ext4 
> mkfs -t ext4 /dev/vdb

![format.down.png-32.3kB][2]

挂载硬盘：
> mount /dev/vdb /hksdata



  [1]: http://static.zybuluo.com/Great-Chinese/4uancys1x2waqyram376idc1/linux%E6%8C%82%E8%BD%BD%E7%A1%AC%E7%9B%98.png
  [2]: http://static.zybuluo.com/Great-Chinese/adrl2byfdn5ara20opapotgc/format.down.png