---
title: hadoop之hive安装
date: 2017-07-21
categories: Bigdata
tags: [hive,bigdata]
comments: false

---
## 环境
- CentOS6.8
- java1.8
- hadoop2.7.2
- mysql5.6.37
- hive2.1.1

## 安装MySQL
- [mysql5.6.37rpm下载地址]()
- 安装  
	1. 安装依赖包: `rpm -ivh MySQL-devel-5.6.37-1.el6.x86_64.rpm`
	2. 安装服务端: `rpm -ivh MySQL-server-5.6.37-1.el6.x86_64.rpm`  
	3. 安装客户端: `rpm -ivh MySQL-client-5.6.37-1.el6.x86_64.rpm`
- 开启服务: `service mysql start`
- 获取登录密码  
mysql装好后会创建一个默认密码文件在root用户目录下: `cat .mysql_secret`
- 登录: `mysql -uroot -p`
- 创建hive用户并授权,允许远程登录

```
CREATE USER 'hive' IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%' IDENTIFIED BY 'hive' WITH GRANT OPTION;
flush privileges;
```
- 创建hive数据库
`create database hive;`

## hive安装及配置

### 下载安装

- [下载地址](http://pan.baidu.com/s/1dFndtKl)
- 解压后移动到/usr/local/下
- 配置环境变量

### 配置hive-env.sh

```
HADOOP_HOME=/usr/local/hadoop
export HIVE_CONF_DIR=/usr/local/hive/conf
export HIVE_AUX_JARS_PATH=/usr/local/hive/lib
```

### 修改hive-site.xml
复制一份hive-default.xml.template为hive-site.xml,内容为:

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://master:3306/hive?createDatabaseIfNotExist=true</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>hive</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>123456</value>
	</property>
	<property>
		<name>hive.metastore.warehouse.dir</name>
		<value>/user/hive/warehouse</value>
	</property>
	<property>
		<name>hive.exec.scratchdir</name>
		<value>/tmp/hive</value>
	</property>
	<property>
		<name>system:java.io.tmpdir</name>
		<value>/usr/local/hive/iotmp</value>
	</property>
	<property>
		<name>hive.exec.local.scratchdir</name>
		<value>/usr/local/hive/iotmp/${user.name}</value>
	</property>
	<property>
		<name>hive.downloaded.resources.dir</name>
		<value>/usr/local/hive/iotmp/${hive.session.id}_resources</value>
	</property>
	<property>
		<name>hive.server2.logging.operation.log.location</name>
		<value>/usr/local/hive/iotmp/${system:user.name}/operation_logs</value>
	</property>
	<property>
		<name>hive.querylog.location</name>
		<value>/usr/local/hive/iotmp/${system:user.name}</value>
	</property>
</configuration>
```

### 在本地新建目录
`mkdir /usr/local/hive/iotmp`

### 在hdfs新建目录并授权
`hdfs dfs -mkdir -p /user/hive/warehouse`  
`hdfs dfs -mkdir -p /tmp/hive`  
`hdfs dfs -chmod -R 777 /user/hive/warehouse`  
`hdfs dfs -chmod -R 777 /tmp/hive`
### 下载JDBC数据库驱动到`/usr/local/hive/lib`目录下

### 配置子节点
将Hive分别到slave1,slave2上,并配置hive环境变量
`scp -r /usr/local/hive slave1:/usr/local/`  
`scp -r /usr/local/hive slave2:/usr/local/`

## hive启动及测试

### hive初始化
`schematool -initSchema -dbType mysql`

### Hive启动
`nohup hive --service metastore >/dev/null 2>&1 &`  ##启动远程模式,否则只能在本地登录  
`hive`  ##客户端登陆

### 子节点访问
`hive`

### 基本命令操作(和mysql差不多)
1. 先创建一个测试库: `create database test;`
2. 创建student表,并指定字段分隔符为tab键(否则会插入NULL)  
`create table student(id int,name string) row format delimited fields terminated by '\t';`
3. 查看下表结构: `describe student;`
4. 创建数据文件,键值要以tab键空格:

	```
	$ cat student.txt 
	1	zhangsan
	2	lisi
	3	wangwu
	```
5. 从本地文件中导入数据到Hive表  
`load data local inpath '/root/student.txt' overwrite into table student;`
6. 从HDFS中导入数据到Hive表  
`load data inpath '/student.txt' overwrite into table student;`
7. 查询是否导入成功
`select * from student;`