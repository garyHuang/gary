# linux ftp


标签（空格分隔）： ftp

[toc]

---
##关于ubuntu14.04.2 LTS搭建ftp服务
######2015/7/23 10:22:29 
###环境：
**Ubuntu 14.04.2**   
**IP：192.168.6.23:**
###工具：
**vsftpd 3.0.2-1**
###安装VSFTPD步骤：
#####1.安装相关的软件包
>*$apt-get install vsftpd*
#####2.修改配置文件/etc/vsftpd.conf
**修改配置文件详细介绍如下:**     
**用户登录控制：**   

    anonymous_enable=YES #允许匿名用户访问ftp服务器。

    no_anon_password=YES #匿名用户登录时不需要输入密码。

    local_enable=YES #允许本地用户访问：ftp localhost。
    
    deny_email_enable=YES #可以创建一个文件保存某些匿名电子邮件的黑名单，以防止这些人使用Dos攻击。
    
    banned_email_file=/etc/vsftpd/banned_emails #保存电子邮件黑名单的目录（默认）
    
**用户权限控制：**
    
    write_enable=YES #开启全局上传
    
    local_umask=022 #本地文件上传的umask设置为022，系统默认。
    
    anon_upload_enable=YES #允许匿名用户上传，当然要在write_enable=YES的情况下。同时必须建立一个允许ftp用户读写的目录。
    
    anon_mkdir_write_enable=YES #允许匿名用户创建目录
    
    chown_uploads=YES #匿名用户上传的文件属主转换为别的用户，一般建议为root。
    
    chown_username=whoever #改此处的whoever为要转换的属主，建议root
    
    chroot_list_enable=YES #用一个列表来限定哪些用户只能在自己目录下活动。
    
    chroot_list_file=/etc/vsftpd/chroot_list #指定用户列表文件
    
    nopriv_user=ftpsecure #指定一个安全账户，让ftp完全隔离和没有特权的账户
    
    其他的建议不要配置。
    
**用户连接和超时设置：**
    
    idle_session_timeout=600 #默认的超时时间
    data_connection_timeout=120 #设置默认数据连接的超时时间
    local_root=/home/linuxidc/公共的/FTP共享文件#设置锁定一个目录（“公共的/FTP共享文件”这个目录位置为自己所创建的）
    
**服务器日志和欢迎信息:**   

    dirmessage_enable=YES#允许为配置目录显示信息  
    ftpd_banner=Welcome to blah FTP service.#ftp的欢迎信息  
    xferlog_enable=YES#打开日志记录功能   
    xferlog_file=/var/log/xferlog  日志记录文件的位置  
#####3.开启vsftpd服务
>*$service vsftpd start*    
#####**关于ftp用户**     
**在ftp中访问ftp服务端的方式：**   
1、ftp服务器上自身的用户可以在客户端访问ftp服务端    
>*#mkdir /home/ftp* #创建ftp根目录                                                                      
>*#useradd  -d   /home/ftp -s /sbin/nologin ftpuser*         #创建用户(注意家目录和非登录)         
>*#passwd  ftpuser*     #设置用户密码                                                             
>*#chown -R ftpuser  /home/ftp*                                             
>*#chmod 755 -R /home/ftp *          #改变ftp根目录属主和权限        
>*#echo "ftpuser" >> /etc/vsftpd.user_list*      #把ftpuser加入到允许访问的队伍        
       
2、使用虚拟用户去访问（虚拟用户是通过映射在本地用户），它有两种创建方式（1.本地数据文件 2.数据库服务器）    
3、使用匿名用户（annoymous），但是需要在vsftp.conf做配置。并可以让匿名用户不用输入密码
###关于虚拟用户的创建
#####1.添加虚拟机用户口令文件
>*$sudo vim /etc/vsftpd/vsftpduser.txt* #这个文件使用与存放用户的信息，然后制作成口令文件  
>**内容如下：**     
    snail#虚拟用户的用户名     
    snail1024#虚拟用户的密码   
	...如有添加其他的用户如上方式...
#####2.生成虚拟用户口令认证文件
将刚添加的vsftpuser.txt虚拟用户口令文件转换成系统识别的口令认证文件，但是首先需要查看系统有没有安装生成口令认证文件所需的软件db-util
>*$sudo dpkg -s db-util*    #检查是否安装db-util    
>*$sudo apt-get install db-util*#安装db-util工具

使用db_load命令去生成虚拟用户口令文件
>*$db_load -T -t hash -f /etc/vsftpd/vsftpduser.txt /etc/vsftpd/vsftpduser.db*#记住：vsftpduser。db这个名字与路径，因为要写入pam.d文件里的vsftpd文件中。否则会报出530的错误提示。并且要对照一下vsftpduser.txt的文件路径，不可写错。
#####3.编辑vsftpd的PAM认证文件
在/etc/pam.d目录下（一般为vsftpd文件，但是也可以自己创建。自己创建的话就要注意记得在vsftpd.conf配置文件中添加此文件名的正确参数，后面会有虚拟用户的vsftpd.conf配置参数）
>*$sudo vim /etc/pam.d/vsftpd*   

    auth required /lib/security/pam_userdb.so db=/etc/vsftpd/vsftpduser
    account required /lib/security/pam_userdb.so db=/etc/vsftpd/vsftpduser
注意：    
1.pam_userdb.so的路径可能不是本文中的路径，此时需要通过find / -name pam_userdb.so去搜索这个动态链接库文件的路径       
2./etc/vsftpd/vsftpduser这个文件是对应生成的/etc/vsftpd/vsftpduser.db文件的路径与名字只是少了.db的后缀    
#####4.建立本地映射用户并设置宿主主目录权限   
所有的FTP虚拟用户需要使用一个系统用户，这个系统用户不需要密码。   
>*$useradd -d /home/vsftpd -s /sbin/nologin ftpuser*   
>*$chmod 700 /homevsftpd*   
#####5.配置vsftpd.conf(设置虚拟用户配置项)
>*$sudo vim /etc/vsftpd.conf    

    guest_enable=YES #开启虚拟用户
    guest_username=vftpuser #FTP虚拟用户对应的系统用户
    pam_service_name=vsftpd #PAM认证文件
    user_config_dir=/etc/vftpuser#虚拟用户所在的目录

#####6.重启vsftpd服务
>*$service vsftpd restart
#####7.建立各个虚拟用户自身的配置文件    
编辑snail用户的配置文件：
>*$sudo vim /etc/vsftpuser/snail

    write_enable=YES #开放snail的写权限
    anon_world_readable_only=NO #开放snail的下载权限
    anon_upload_enable=YES #开放snail的上传权限
    anon_mkdir_write_enable=YES #开放snail创建目录的权限
    anon_other_write_enable=YES #开放snail删除和重命名的权限
#####8.测试虚拟用户登录FTP
    C:\User\Administrator>ftp 192.168.6.23  
    连接到192.168.6.23。
    220 Welcome to BOB FTP server   
    用户(192.168.120.240(none)):snail   
    331 Please specify the password.   
    密码:  
    230 Login successful.