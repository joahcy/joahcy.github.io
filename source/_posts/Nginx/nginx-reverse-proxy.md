---
title: nginx反向代理配置
date: 2017-06-01
categories: Nginx
tags: [Nginx,反向代理]

---

## 修改host文件(来实现域名访问)
```
192.168.170.128 8081.zyx.com
192.168.170.128 8082.zyx.com
```

## 安装两个tomcat实例并修改端口

## 修改nginx配置文件
```
upstream tomcatserver1 {
	server 192.168.170.128:8080;
}

upstream tomcatserver2 {
	server 192.168.170.128:8081;
}

server {
    listen       80;
    server_name  8081.zyx.com;

    location / {
        proxy_pass   http://tomcatserver1;
        index  index.html index.htm;
    }
}

server {
    listen       80;
    server_name  8082.zyx.com;

    location / {
        proxy_pass   http://tomcatserver2;
        index  index.html index.htm;
    }
}
```