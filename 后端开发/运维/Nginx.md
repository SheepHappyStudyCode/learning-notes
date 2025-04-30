﻿﻿﻿﻿# Nginx
## 1 安装部署
使用docker
## 2 基本使用
### 2.1 最小配置解析
1. worker_processers：工作线程数
2. include：包含其他配置文件
### 2.2 反向代理
在server的location下添加：`proxy_pass 被代理服务器的ip | 域名;`
### 2.3 负载均衡
1. 配置upstream，里面写上被代理服务器的ip和相应的权值。         
2. 被代理服务器的属性
	- down：停机
	- backup：备用             

3. 配置示例：

```nginx
http {
    upstream server1{
        server 127.0.0.1:8080 weight=3;
        server 127.0.0.1:8081 weight=2;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://server1;
        }
    }
}

```

### 2.4 动静分离
1. 适用于中小企业
2. 写法：多配置几个location
### 2.5 URL rewrite
### 2.6 网关服务器
### 2.7 防盗链
### 2.8 高可用场景及解决方案
设置一个主服务器和一个备用机，二者用keepalive进行通信，同时有一个虚拟ip在两台服务器进行切换。
### 2.9 Https证书配置

### 2.10 虚拟主机配置

一台 NGINX 可以部署个网站，使用不同的域名作区分

```nginx
http {
    server {
        listen 80;
        server_name example.com www.example.com;
        root /var/www/example.com;
        
        location / {
            index index.html index.htm;
        }
    }
    
    server {
        listen 80;
        server_name another-example.org www.another-example.org;
        root /var/www/another-example.org;
        
        location / {
            index index.html index.htm;
        }
    }
}
```



## 3 常用配置

```nginx
# 防止页面刷新出现 404
location / {
  try_files $uri $uri/index.html /index.html;
}
# 反向代理
location /api {
        proxy_pass http://127.0.0.1:8123;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_buffering off;
        proxy_set_header Connection "";
    }
    
location /api/ws {
    proxy_pass http://127.0.0.1:8123;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_buffering off;
    proxy_read_timeout 86400s;
}
# 代理图片 
location ^~ /files/ {
    alias /techhub/; // alias指令会用指定的路径替换匹配的位置部分，然后将剩余的请求路径附加到这个新路径后面。
}
```



