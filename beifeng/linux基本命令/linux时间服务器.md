# linux时间服务器

标签（空格分隔）： linux基本命令

[TOC]

---

linux时区设置设置Asia/Shanghai东8区


拷贝上海时区文件到/etc 目录下
sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime/



##1、使用命令
```
cat /etc/*release
```
![time001.png-6.5kB][1]

##2、确认是否安装ntpd
```
rpm -qf /usr/sbin/ntpd
```
![installntpd.png-2.5kB][2]
##3、sudo service ntpd start
![starttimeserver.png-10kB][3]

##挂载linux 硬盘
查看未挂载的硬盘命令：fdisk -l 
挂


  [1]: http://static.zybuluo.com/Great-Chinese/76q5m5kupetn39ujfmesl6qw/time001.png
  [2]: http://static.zybuluo.com/Great-Chinese/gxnpl30pquqao2oprpbpgwwv/installntpd.png
  [3]: http://static.zybuluo.com/Great-Chinese/x52ga2m3opxfh99u94dm8bdd/starttimeserver.png