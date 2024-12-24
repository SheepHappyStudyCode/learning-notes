# Nginx
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
### 2.4 动静分离
1. 适用于中小企业
2. 写法：多配置几个location
### 2.5 URL rewrite
### 2.6 网关服务器
### 2.7 防盗链
### 2.8 高可用场景及解决方案
设置一个主服务器和一个备用机，二者用keepalive进行通信，同时有一个虚拟ip在两台服务器进行切换。
### 2.9 Https证书配置

