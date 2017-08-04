---
title: nginx配置虚拟主机
date: 2017-06-01
categories: Nginx
tags: [Nginx,虚拟主机]

---

## 基于IP的虚拟主机
- 1.配置多IP
- 2.nginx的配置文件在http节点下加一个server节点,如: 
 
```
server {
    listen       80;
    server_name  192.168.170.128; \# 配置IP

location / {
    root   html-128;
    index  index.html index.htm;
}

server {
    listen       80;
    server_name  192.168.170.100; \# 配置IP

location / {
    root   html-100;
    index  index.html index.htm;
}
```
- 3.复制html页面
- 4.访问测试

## 基于端口的虚拟主机
- 1.添加server节点,如:

```
server {
    listen       81; \# 配置端口
    server_name  192.168.170.128;

location / {
    root   html-81;
    index  index.html index.htm;
}

server {
    listen       82; \# 配置端口
    server_name  192.168.170.128;

location / {
    root   html-82;
    index  index.html index.htm;
}
```
- 2.复制html页面
- 3.访问测试

## 基于域名的虚拟主机
**使用最多的虚拟主机配置方式,一个域名只能绑定一个IP，一个IP可以被多个域名绑定**
- 1.修改host文件实现域名访问
	如：
		192.168.170.128  www.zyx.com
		192.168.170.128  blog.zyx.com
- 2.修改nginx的配置文件

```
server {
	listen       80;
	server_name  www.zyx.com;

location / {
    root   html-www;
    index  index.html index.htm;
}

server {
    listen       80;
    server_name  blog.zyx.com;

location / {
    root   html-blog;
    index  index.html index.htm;
}
```