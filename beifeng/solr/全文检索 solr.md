# 全文检索 solr

标签（空格分隔）： solr

[toc]

---
##1、安装solr
1.1 下载需要安装的solr版本
http://archive.apache.org/dist/lucene/solr/
1.2快速安装文档
http://lucene.apache.org/solr/quickstart.html
1.3其他说明文档
http://lucene.apache.org/solr/resources.html


##2、solr tomcat 安装
2.1 解压solr-x.x.x.zip
![QQ截图20160305200625.jpg-18.3kB][1]
2.2进入 拷贝文件 dist/solr-x.x.x.war 到 tomcat_home/webapps 目录下，并改名为 solr.war

2.3启动 tomcat，让solr.war自动解压，然后 删除solr.war 文件

2.4 拷贝 example\lib\ext\ 里面的jar到 tomcat_home\lib 目录下

2.5拷贝 example\multicore 到 一个指定的目录，我这里拷贝到tomcat_home目录下，并改名为  solrhome

2.6 修改 tomcat_home\webapps\solr\WEB-INF\web.xml 里面solrhome 指向 刚刚 的目录
```
<env-entry>
   <env-entry-name>solr/home</env-entry-name>
   <env-entry-value>D:\\JAVA\\tomcat8\\solrhome\\</env-entry-value>
   <env-entry-type>java.lang.String</env-entry-type>
</env-entry>
```
2.7启动tomcat
访问 http://127.0.0.1:8080/solr/  看到入邪界面，说明配置成功
![solr002.jpg-61.1kB][2]









  [1]: http://static.zybuluo.com/Great-Chinese/hn38iz1j1otqgymq4vcj2b87/QQ%E6%88%AA%E5%9B%BE20160305200625.jpg
  [2]: http://static.zybuluo.com/Great-Chinese/oywb8gjad7g3zp91wuinsbz5/solr002.jpg
  
  