---
title: Linux下安装JDK
date: 2017-06-02
categories: Java
tags: [JDK,Java]
---
## 去官网下载自己需要的linux的JDK版本
## 解压jdk到当前目录  
`tar -zxvf jdk-8u60-linux-x64.tar.gz`  
得到文件夹 jdk1.8.0_60

## 安装完毕为它建立一个链接以节省目录长度
(我没用这一步)
`ln -s /usr/java/jdk1.8.0_60/ /usr/jdk`

## 编辑配置文件，配置环境变量
`vim /etc/profile`
添加如下内容：JAVA_HOME根据实际目录来
```
JAVA_HOME=/usr/java/jdk1.8.0_60
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```

## 重启机器或执行命令:  
`source /etc/profile`或者`sudo shutdown -r now`

## 查看安装情况
`java -version`
得到以下信息就说明安装成功
```
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) Client VM (build 25.60-b23, mixed mode)
```