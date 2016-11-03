# nginx+tomcat集群

标签（空格分隔）： nginx+tomcat

---

```
#user  nobody;
worker_processes  1;
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
	#配置 tomcat主机
	upstream localhost{
		server 127.0.0.1:908;
    }
    
    server {
        listen       80;
        server_name  localhost;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
		send_timeout 600;
		client_body_timeout 600;
		client_header_timeout 600;
		error_page 400 = /404.html; 
		error_page 404 = /404.html; 
        location / {
			# 禁止非 post 和get的请求，除非网站需要
			if ($request_method = DELETE ) {
				return 403;
			}
			if ($request_method = OPTIONS ) {
				return 403;
			}
			if ($request_method = PUT ) {
				return 403;
			}
			if ($request_method = TRACE ) {
				return 403;
			}
			if ($request_method = CONNECT ) {
				return 403;
			}
            root   html;
            index  index.html index.htm;
            # nginx解析的地址
			proxy_pass http://localhost/ ;
			#这个必需配置，否则tomcat将取不到访问的serverName和端口
			proxy_set_header   Host             $host;
			proxy_set_header   X-Real-IP        $remote_addr;
			
			proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
    }
}

linux开放某个端口 外部访问
```
iptables -I INPUT -p tcp --dport 8888 -j ACCEPT
```



