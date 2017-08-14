---
title: spark集群安装
date: 2017-07-22
categories: Bigdata
tags: [spark,bigdata]
comments: false

---
## 环境

- scala2.11.5
- java1.8(scala这个版本要求java1.8,所以我把之前的java1.7换成1.8了,[1.8下载地址](http://pan.baidu.com/s/1mimPhaC)) 
- spark2.1.0

## 安装Scala

- 确认已经装好了jdk
`java -version`
- [下载地址](http://pan.baidu.com/s/1kVmRoqV)
- 解压并移动到/usr/local/scala
- 配置环境变量

## 安装spark

### 确认已经装好hadoop
`hadoop version`

### 下载
[下载地址](http://pan.baidu.com/s/1pKB5tpD)

### 解压配置环境变量
解压并移动到/usr/local/spark  
然后配置环境变量

### 到这我们可以简单使用spark

在`$SPARK_HOME/examples/src/main`目录下有一些 Spark 的示例程序,有 Scala,Java,Python,R 等语言的版本.我们可以先运行一个示例程序 SparkPi(即计算 π 的近似值),执行如下命令:  
`run-example SparkPi`  
然后就可以看到结果.

### 修改配置文件
- 复制spark-env.sh.template成spark-env.sh

```
HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)
SPARK_LOCAL_IP=192.168.170.170
SPARK_MASTER_HOST=master
SPARK_WORKER_CORES=2
SPARK_WORKER_INSTANCES=1
SPARK_WORKER_DIR=/usr/local/spark/work 
```
- 复制slaves.template成slaves

```
master
slave1
slave2
```

### 将配置好的spark文件复制到Slave1和Slave2节点。
`scp /usr/local/spark slave1:/usr/local`  
`scp /usr/local/spark slave2:/usr/local`
修改Slave1和Slave2配置
在Slave1和Slave2上分别配置Spark的环境变量.  
在Slave1和Slave2修改spark-env.sh,将SPARK_LOCALIP改成Slave1和Slave2对应节点的IP.

### 在Master节点启动集群
`$SPARK_HOME/sbin/start-all.sh`

### 用jps命令查看集群是否启动成功
- Master在Hadoop的基础上新增了: Master
- Slave在Hadoop的基础上新增了: Worker

### 在浏览器上查看
`http://master:8080`
看到下图说明安装成功
![spark-cluster](http://oqwn6kueb.bkt.clouddn.com//blog/spark-cluster.jpg)