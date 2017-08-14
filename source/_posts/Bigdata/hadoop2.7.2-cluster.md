---

title: hadoop2.7.2集群搭建
date: 2017-07-20
categories: Bigdata
tag: [hadoop,cluster,bigdata]
comments: false

---

## 前言
搭建hadoo集群是学习大数据不可或缺的一步,然而自己练习又没有那么多台主机,只能在虚拟机上搭建了,自己也是第一次搭建,所以在这进行一些必要步骤的记录,下面开始吧.  
**官网下载软件很慢,所以下面提供的都是我的百度云盘下载地址**

## 环境
- CentOS6.8
- java1.7
- hadoop2.7.2

## 安装vmware
[vmware10.0下载地址](http://pan.baidu.com/s/1nvgb2bv),然后就是傻瓜式安装,这里就不详细说了.

## 安装linux(我用的是CentOS6.8)
[CentOS6.8下载地址](http://pan.baidu.com/s/1o7TimL0), 如果自己不需要设置分区的话,基本就是下一步下一步完成安装.具体google,百度也有很多教程.  
安装完成后对网络进行设置,使用nat模式,点击安装好的虚拟机,右键设置,选择网络适配器,然后选择nat模式.启动虚拟机进行登录.  
查看IP并验证是否能上网,分别执行以下操作:  
`ifconfig`  eth0 那块网卡的addr就是nat模式下的ip  
`ping www.baidu.com`  虽然百度害人不浅,但在验证网络上还是有一点用处的,哈哈~~~  
系统默认DHCP动态分配IP,当系统每次重启,IP可能会变,所以我们要手动设置静态ip.  
编辑网卡文件:  
`vim /etc/sysconfig/network-scripts/ifcfg-eth0`  
我的设置如下:  

```
DEVICE=eth0  ## 网卡名
TYPE=Ethernet
ONBOOT=yes  ## 设置为开机启动
BOOTPROTO=static
HWADDR=00:0C:29:65:9C:41 
IPADDR=192.168.170.170  ## NAT模式下静态IP
PREFIX=24  ## 和子网掩码对应
GATEWAY=192.168.170.2  ## 网关
DNS1=192.168.170.2  ## DNS
```
设置完成后重启网络:  
`service network restart`  
然后查看IP,并ping一下百度验证是否能上网.若不行,仔细查看配置信息是否正确.

## 添加hadoop用户
1. `user add hadoop`
2. `passwd hadoop`
3. 登录: `su hadoop`

## 安装JDK
hadoop的运行依赖java所以得先安装JDK.
### 卸载OpenJDK
CentOS6.8自带OpenJDK,安装之前先卸载OpenJDK.  

- 查看当前安装的OpenJDK  
`rpm -qa | grep jdk`  
得到以下信息  
`java-1.7.0-openjdk.x86_64 1:1.7.0.99-2.6.5.1.el6`  
- 使用yum命令卸载
`yum -y remove java-1.7.0-openjdk.x86_64 1:1.7.0.99-2.6.5.1.el6`

### 安装SunJDK
- [JDK1.7下载地址](http://pan.baidu.com/s/1qXCjosO)
- 在linux上下载或从windows上传到linux上.[FTP工具](http://pan.baidu.com/s/1hs3Spb2)
- 解压jdk  
`tar -zxvf jdk-7u79-linux-x64.tar.gz`
- 将解压的得到的jdk1.7.0_79目录移动到`/usr/local`目录下并重命名为java1.7
`mv jdk1.7.0_79 /usr/local/java1.7`
- 配置环境变量
在当前用户家目录执行: `vim .bashrc`  
添加以下内容:
```
export JAVA_HOME=/usr/local/java1.7
export CLASSPATH=.:$JAVA_HOME/lib/
export PATH=$PATH:$JAVA_HOME/bin
```
- 让环境变量生效
`source .bashrc`
- 验证是否配置成功
`java -version`  
得到以下信息说明配置成功了
```
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
```

## 安装hadoop
- [hadoop2.7.2下载地址](http://pan.baidu.com/s/1jHDbyiq)
- 解压并移动到/usr/local目录下
`tar -zxvf hadoop-2.7.2.tar.gz`  
`mv hadoop-2.7.2 /usr/local/hadoop`
- 配置环境变量  
`vim .bashrc`  
java和hadoop最终配置如下:
```
export JAVA_HOME=/usr/local/java1.7
export CLASSPATH=$JAVA_HOME/lib/
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$JAVA_HOME/bin/:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
```
然后 `source .bashrc`
- 查看是否配置成功
`hadoop version`  
得到以下信息说明配置成功
```
Hadoop 2.7.2
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r b165c4fe8a74265c792ce23f546c64604acf0e41
Compiled by jenkins on 2016-01-26T00:08Z
Compiled with protoc 2.5.0
From source with checksum d0fda26633fa762bff87ec759ebe689c
This command was run using /usr/local/hadoop/share/hadoop/common/hadoop-common-2.7.2.jar
```

## 配置Hadoop分布式集群

### 修改hostname为master
`vim /etc/sysconfig/network`  
改为: `HOSTNAME=master`

### 设置hosts文件使主机名和IP地址相对应
`vim /etc/hosts`  
添加以下信息
```
192.168.170.170 master  ## 本机IP
192.168.170.171 slave1  ## 一会要设置的从节点IP
192.168.170.172 slave2
```
**其他两台slave的主机也修改对应的slave, slave2.如果修改完主机名后没有生效.那么重启系统便可以.三台机子的主机名与hosts均要修改**

### 在hadoop文件夹下建立如下四个文件夹
- 目录/tmp, 用来存储临时生成的文件
- 目录/hdfs, 用来存储集群数据
- 目录hdfs/data, 用来存储真正的数据
- 目录hdfs/name, 用来存储文件系统元数据
`mkdir tmp hdfs hdfs/data hdfs/name`

### 配置hadoop文件
在hadoop/etc/hadoop目录下,主要配置以下7个文件:
- slaves 
- hadoop-env.sh
- yarn-env.sh
- core-site.xml
- mapred-site.xml
- hdfs-site.xml
- yarn-site.xml

#### 修改slaves文件
修改为:

```
slave1
slave2
```

#### 修改hadoop-env.sh文件
将 `export JAVA_HOME=${JAVA_HOME}` 那一行改为 `export JAVA_HOME=/usr/local/java1.7` 

#### 修改yarn-env.sh文件
同样将 `export JAVA_HOME=${JAVA_HOME}` 那一行改为 `export JAVA_HOME=/usr/local/java1.7` 

#### 修改core-site.xml文件
配置为:

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop/tmp</value>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>
</configuration>
```

#### 修改mapred-site.xml文件
先把mapred-site.xml.template文件复制一份为mapred-site.xml再修改.  
配置为:

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>master:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>master:19888</value>
    </property>
</configuration>
```

#### 修改hdfs-site.xml文件

```
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/hdfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>master:9001</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
</configuration>
```

#### 修改yarn-site.xml文件
```
<configuration>
    <property>
		<name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>master:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>master:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>master:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>master:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>master:8088</value>
    </property>
</configuration>

```

## 关闭防火墙和SELinux
切记一定要关闭,否则Hadoop启动会出错.

- 关闭防火墙  
查看防火墙状态: `service iptables status`  
若未关闭,则执行: `service iptables stop`
- 关闭SELinux  
查看SELinux状态: `getenforce`  
关闭SELinux: `setenforce 0`  **系统重启后还会生效**  
永久关闭: `vim /etc/selinux/config`  将selinux状态改为`disabled`

## 安装并配置从节点
1. 再安装两个虚拟机,重复以上步骤进行配置,这样有点麻烦也浪费时间,所以我们可以直接复制master节点的文件为两份,分别为slave1和slave2,这样就不用重复配置了,省事不少,然后在vmware里面打开.
2. 因为从节点是复制的,所以网卡信息也复制了,将会导致从节点上不了网.因此,我们要在vmware里面点击打开的slave1节点,点击设置,选中网络适配器将其移除,然后再添加一块新的网络适配器(网卡).**注意也要选择nat模式**, slave2同理.然后同时启动slave1和slave2.  
3. 启动后设置slave1节点IP为之前master设定好的IP: `192.168.170.171`  
slave2节点IP为: `192.168.170.172`
4. 修改hostname,将slave1和slave2节点的hostname分别改为slave1和slave2,参考master修改hostname.

## 实现免密码登陆
hadoop各个节点之间需要数据通信,为了方便操作,在数据通信时,不要我们再逐台输入密码.我们需要配置免密码登录的ssh.
  
1. 使用以下命令在三台虚拟机上分别生成各自的公钥,私钥.
`ssh-keygen -t rsa`  
**全部按回车进行生成,默认生成位置在家目录下的.ssh文件夹**
2. 将master的公钥追加到authorized_keys中.  
`cat .ssh/id_rsa.pub >> .ssh/authorized_keys`
3. 将slave1和slave2公钥追加到master的authorized_keys文件中.  
在两台从节点都执行: `ssh-copy-id -i .ssh/id_rsa.pub master`
4. 将master的authorized_keys复制到两个从节点上  
`scp .ssh/authorized_keys slave1:~/.ssh/`  
`scp .ssh/authorized_keys slave2:~/.ssh/`
5. 测试
在三台虚拟机上用ssh命令进行互相测试：ssh master, ssh slave1, ssh slave2.  
以slave1为例:  
```
[hadoop@slave1 ~]$ ssh master
Last login: Thur Jul 20 13:16:39 2017 from slave2
[hadoop@slave1 ~]$ ssh slave2
Last login: Thur Jul 20 13:17:21 2017 from slave1
```
不需要密码,是不是方便多了呢。

## 格式化hdfs文件系统
OK,到这,我们就算配置完成了.不过,我们要想使用hadoop,还需要格式化hdfs文件系统.
hadoop2.7.2版本的格式化命令也有变化,之前用hadoop namenode -format,现在是hdfs.  
执行: `hdfs namenode -format`  
执行上面的命令后,在后面的提示中如果看到successfully formatted的字样,说明hdfs格式化成功! 

## 启动集群
**所有启动关闭脚本在`/usr/local/hadoop/sbin`目录下**  
之前版本使用 `start-all.sh` 一个脚本启动,但是我想看看它到底启动了什么,查看源代码后发现脚本里有这么一句话: "This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh",表明已经过时了,要分别用start-dfs.sh 和 start-yarn.sh来启动.  

- 启动dfs并查看进程
```
[hadoop@master ~]$ start-dfs.sh
17/07/20 13:42:52 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [master]
master: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hadoop-namenode-master.out
slave1: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hadoop-datanode-slave1.out
slave2: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hadoop-datanode-slave2.out
Starting secondary namenodes [master]
master: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-hadoop-secondarynamenode-master.out
17/07/20 13:45:34 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
[hadoop@master ~]$ jps
27742 SecondaryNameNode
27885 Jps
27574 NameNode
```
出现的警告由于glibc版本(系统的是2.12, hadoop需要的是2.14)问题引起的,解决办法:   
在`/usr/local/hadoop/etc/hadoop/log4j.properties` 文件最后添加: `log4j.logger.org.apache.hadoop.util.NativeCodeLoader=ERROR` 即可解决.

- 启动yarn并查看进程
```
[hadoop@master ~]$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-resourcemanager-master.out
slave2: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-nodemanager-slave2.out
slave1: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-nodemanager-slave1.out
[hadoop@master ~]$ jps
28053 ResourceManager
27742 SecondaryNameNode
28122 Jps
27574 NameNode
```

- 在从节点查看进程
```
[hadoop@slave1 ~]$ jps
4603 DataNode
4932 Jps
4785 NodeManager
```
```
[hadoop@slave2 ~]$ jps
5962 Jps
5674 DataNode
5830 NodeManager
```
三台虚拟机状态都正常.

## 测试

### 在浏览器输入网址查看
输入 `192.168.170.170:8088` 或者 `192.168.170.170:50070` 能看到网页信息说明启动成功,可以查看节点信息.

### 检验hadoop可用性
单纯的看进程和网页信息还不能完全的说明hadoop可用,我们要使用hadoop提供的 `hadoop-mapreduce-examples-2.7.2.jar` 包的wordcount功能检验一下hadoop是不是真的可以进行数据的分析.

#### 准备工作
- 在hdfs文件系统创建input文件夹: `hdfs dfs -mkdir /input`
- 上传需要分析处理的数据文件: `hdfs dfs -put /usr/local/hadoop/*.txt /input`
- 启动historyserver,不然下一步会报以下错误信息:

```
Caused by: java.net.ConnectException: 拒绝连接
    at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
    at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:739)
    at org.apache.hadoop.net.SocketIOWithTimeout.connect(SocketIOWithTimeout.java:206)
    at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:531)
    at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:495)
    at org.apache.hadoop.ipc.Client$Connection.setupConnection(Client.java:614)
    at org.apache.hadoop.ipc.Client$Connection.setupIOstreams(Client.java:712)
    at org.apache.hadoop.ipc.Client$Connection.access$2900(Client.java:375)
    at org.apache.hadoop.ipc.Client.getConnection(Client.java:1528)
    at org.apache.hadoop.ipc.Client.call(Client.java:1451)
    ... 33 more
```
启动: `mr-jobhistory-daemon.sh start historyserver`

#### 开始运行mapreduce
执行: `hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /input /output`  
具体如下:

```
[hadoop@master ~]$ hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /test/input /test/output
17/07/20 15:35:55 INFO client.RMProxy: Connecting to ResourceManager at master/192.168.170.170:8032
17/07/20 15:36:05 INFO input.FileInputFormat: Total input paths to process : 3
17/07/20 15:36:06 INFO mapreduce.JobSubmitter: number of splits:3
17/07/20 15:36:09 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1502431075552_0003
17/07/20 15:36:13 INFO impl.YarnClientImpl: Submitted application application_1502431075552_0003
17/07/20 15:36:13 INFO mapreduce.Job: The url to track the job: http://master:8088/proxy/application_1502431075552_0003/
17/07/20 15:36:13 INFO mapreduce.Job: Running job: job_1502431075552_0003
17/07/20 15:38:39 INFO mapreduce.Job: Job job_1502431075552_0003 running in uber mode : false
17/07/20 15:38:39 INFO mapreduce.Job:  map 0% reduce 0%
17/07/20 15:42:34 INFO mapreduce.Job:  map 44% reduce 0%
17/07/20 15:42:39 INFO mapreduce.Job:  map 78% reduce 0%
17/07/20 15:42:40 INFO mapreduce.Job:  map 89% reduce 0%
17/07/20 15:42:42 INFO mapreduce.Job:  map 100% reduce 0%
17/07/20 15:44:08 INFO mapreduce.Job:  map 100% reduce 100%
17/07/20 15:44:18 INFO mapreduce.Job: Job job_1502431075552_0003 completed successfully
17/07/20 15:44:20 INFO mapreduce.Job: Counters: 49
        File System Counters
                FILE: Number of bytes read=12989
                FILE: Number of bytes written=495819
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=17212
                HDFS: Number of bytes written=8983
                HDFS: Number of read operations=12
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
        Job Counters
                Launched map tasks=3
                Launched reduce tasks=1
                Data-local map tasks=3
                Total time spent by all maps in occupied slots (ms)=781665
                Total time spent by all reduces in occupied slots (ms)=59106
                Total time spent by all map tasks (ms)=781665
                Total time spent by all reduce tasks (ms)=59106
                Total vcore-milliseconds taken by all map tasks=781665
                Total vcore-milliseconds taken by all reduce tasks=59106
                Total megabyte-milliseconds taken by all map tasks=800424960
                Total megabyte-milliseconds taken by all reduce tasks=60524544
        Map-Reduce Framework
                Map input records=322
                Map output records=2347
                Map output bytes=24935
                Map output materialized bytes=13001
                Input split bytes=316
                Combine input records=2347
                Combine output records=897
                Reduce input groups=840
                Reduce shuffle bytes=13001
                Reduce input records=897
                Reduce output records=840
                Spilled Records=1794
                Shuffled Maps =3
                Failed Shuffles=0
                Merged Map outputs=3
                GC time elapsed (ms)=31699
                CPU time spent (ms)=34140
                Physical memory (bytes) snapshot=413155328
                Virtual memory (bytes) snapshot=3358351360
                Total committed heap usage (bytes)=364929024
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=16896
        File Output Format Counters
                Bytes Written=8983
```

查看处理后的数据:  
`hdfs dfs -cat /output/part-r-00000`  
大概如下:

```
......
only    4
or      67
or,     1
org.apache.hadoop.util.bloom.*  1
origin  1
original        2
other   9
otherwise       3
otherwise,      3
our     2
out     1
outstanding     1
own     4
owner   4
owner.  1
owner]  1
ownership       2
page"   1
part    4
patent  5
patent, 1
percent 1
perform,        1
performing      1
permission      1
permission.     1
permissions     3
permitted       2
permitted.      1
perpetual,      2
......
```
数据有点多...  
也可以在浏览器访问`192.168.170.170:50070`,点击`Utilities`下面的`Browse the file system`.在里面查看文件,也可以下载后查看.

## 后记
至此,hadoop集群搭建完成,wordcount小例子完美运行.该休息下了 -_-||













