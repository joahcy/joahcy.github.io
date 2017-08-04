---
title: nginx负载均衡配置
date: 2017-06-01
categories: Nginx
tags: [Nginx,负载均衡]
---

## 1、修改host文件(来实现域名访问)
`192.168.170.128 8083.zyx.com`  
`192.168.170.128 8084.zyx.com`

## 2、安装两个tomcat实例并修改端口
## 3、修改nginx配置文件
```
upstream tomcatserver {
	server 192.168.170.128:8083  weight=2;
	server 192.168.170.128:8084  weight=1; \#weight默认为1
}

server {
    listen       80;
    server_name  www.zyx.com;

    location / {
        proxy_pass   http://tomcatserver;
        index  index.html index.htm;
    }
}
```

## 配置权重
 
- 在http节点里添加:
```
\#定义负载均衡设备的 Ip及设备状态 
upstream myServer {   
    server 127.0.0.1:9090 down; 
    server 127.0.0.1:8080 weight=2; 
    server 127.0.0.1:6060; 
    server 127.0.0.1:7070 backup; 
}
```
- 在需要使用负载的Server节点下添加   
`proxy_pass http://myServer;`  

- 每个设备的状态:
	- down: 表示单前的server暂时不参与负载.  
	- weight: 默认为1.weight越大，负载的权重就越大.  
	- max_fails: 允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误.  
	- fail_timeout: max_fails 次失败后，暂停的时间.  
	- backup: 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻.  