# Mysql

标签（空格分隔）： Mysql

[TOC]

---
```
--查询显示当前最大数量数值
show variables like'%max_connections%';
--设置最大链接数
set global max_connections=2000 ;
--立即生效
flush privileges; 
--当前数据库状态
SHOW STATUS;
-- 查询当前使用的链接
show processlist 

```
