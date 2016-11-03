# linux无密钥登录

标签（空格分隔）： linux基本命令

[TOC]

---

##第一步生成本机密钥
```
    ssh-keygen -t  rsa
```
![linux no pwd login.png-19.3kB][1]
第二个红色框框表示，按回车报错密钥到/home/gary/.ssh/id_rsa 文件
第三个红色框框，使用ssh登录的密码，既然是无密码登录，所以直接回车就可以，连续回车两次。

##第二步拷贝密钥
这一步很重要，如果拷贝错误，很可能导致登录不成功。
```
 ssh-copy-id -i ~/.ssh/id_rsa.pub hostname
```
![linux no pwd login02.png-17.6kB][2]
图中标出的四个地方要特别注意，拷贝密钥只能拷贝 ~/.ssh/id_rsa.pub文件，不然回导致无密钥登录失败。

##第三步测试是否成功
```
 ssh hostname
```
 ![linux no pwd login03.png-3.9kB][3]
 如果登录输入密码，则ssh无密码登录操作失败。
 需要到 拷贝的机器上 删除 /home/用户名/.ssh 目录，重新操作。
 
 
普通账号配置sudo：
```
#允许执行的命令目录
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/hksdata/soft/java/jdk1.7.0_67/bin"
#允许使用sudo的账号
zhoujian ALL=(root)NOPASSWD:ALL
ldh ALL=(root)NOPASSWD:ALL
hf ALL=(root)NOPASSWD:ALL
```


  [1]: http://static.zybuluo.com/Great-Chinese/2qcm32juy7yjckoy1rt1sl6x/linux%20no%20pwd%20login.png
  [2]: http://static.zybuluo.com/Great-Chinese/95tmplxhx4vg5kjbo409upzh/linux%20no%20pwd%20login02.png
  [3]: http://static.zybuluo.com/Great-Chinese/pkt5h4qs0mbnt0i0chf794o9/linux%20no%20pwd%20login03.png