---
layout: post
title:  "hadoop伪分布式模式环境搭建"
keywords: "hadoop"
description: "在linux环境搭建Hadoop伪分布式模式"
category: hadoop 
tags: [hadoop]
---
###服务器环境
```
~$ cat /proc/version
Linux version 4.4.0-24-generic (buildd@lcy01-30) (gcc version 5.3.1 20160413 (Ubuntu 5.3.1-14ubuntu2.1) ) #43-Ubuntu SMP Wed Jun 8 19:25:16 UTC 2016
```
###Hadoop安装教程(伪分布式模式)
####JDK安装
```
~$ java -version
java version "1.8.0_92"
Java(TM) SE Runtime Environment (build 1.8.0_92-b14)
Java HotSpot(TM) Server VM (build 25.92-b14, mixed mode)
```
####hadoop安装版本
```
版本命令：hadoop-2.5.2
下载命令：http://mirrors.cnnic.cn/apache/hadoop/common/hadoop-2.5.2/hadoop-2.5.5.tar.gz
```
####创建hadoop用户
#####1.切换到root用户下
```
sudo su root
```
#####2.新增hadoop用户
```
adduser hadoop
passwd hadoop
```
#####3.在sudoers文件中添加hadoop权限
```
vi /etc/sudoers
在root ALL=(ALL:ALL) ALL下面新增 hadoop ALL=(ALL:ALL) ALL
最终结果如下:
root ALL=(ALL:ALL) ALL
hadoop ALL=(ALL:ALL) ALL
```
#####4.免密码ssh设置
```
切换到hadoop用户执行
/usr/bin/ssh-keygen -t rsa --执行完这句，按三次回车即可生成公钥与私钥
cat ~/.ssh /id rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
#####5.验证hadoo用户的ssh免登录认证
```
执行ssh localhost命令
如果没出现 ssh:connect to host localhost port 22:Connection refused信息，说明ssh免登录设置成功，否则看下面的安装过程遇到问题中的解决方案
```
####安装hadoop2.5.2
#####1.将下载的hadoop-2.5.2.tar.gz安装包，解压到hadoop的用户目录（/home/hadoop）
```
tar -xzvf hadoop-2.5.2.tar.gz
```
#####2.在hadoop用户的根目录创建文件夹
```
mkdir -p dfs/name
mkdir -p dfs/data
```
#####3.修改配置文件
######3.1.hadoop可以在单节点上以伪分布式的方式运行，Hadoop进程以分离的Java进程来运行，节点即NameNode也是DataNode。需要修改2个配置文件etc/hadoop/core-site.xml和etc/hadoop/hdfs-site.xml
* core-site.xml修改如下:

```
#Hadoop自升级到2.x版本之后，有很多属性的名称已经被遗弃了，虽然这些被遗弃的属性名称目前还可以用，但是这里还是建议用新的属性名(上面的fs.defaultFS在老版本中使用fs.default.name，现在还是可以用的，但是建议使用新的)
<configuration>  
        <property>  
          <name>fs.defaultFS</name>  
          <value>hdfs://127.0.0.1:9000</value>  
        </property>   
</configuration>  
```
* hdfs-site.xml修改如下:

```
配置说明:添加hdfs的指定URL路径，由于是伪分布模式，所以配置的是本机IP ，可为真实Ip、localhost。主要是对namenode 和 datanode 存储路径的设置。其实默认是存储在file://${hadoop.tmp.dir}/dfs/name和data 下的。所以这里也不需配置的。但默认的是临时文件，重启就没有了，所以我这里还是设置了专门的路径保存
```
* 将mapred-site.xml.template重命名为mapred-site.xml，并添加如下内容(目的是告诉hadoop，MapReduce是运行在yarn这个框架上)

```
<property>  
       <name>mapreduce.framework.name</name>  
        <value>yarn</value>  
</property>
```
* yarn-site.xml修改如下:

```
<property>  
    <name>yarn.nodemanager.aux-services</name>  
    <value>mapreduce_shuffle</value>  
</property>  
```
#####4.为hadoop指定jdk(修改etc/hadoop/hadoop-env.sh文件)

```
export JAVA_HOME=${JAVA_HOME}  --原来
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_92
```
#####5.添加环境变量(sudo vi /etc/profile)
```
sudo vi /etc/profile
export HADOOP_HOME=/home/hadoop/hadoop-2.5.2
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH
```
####启动hadoop
* 切换到hadoop-2.5.2的bin文件夹下首先格式化namenode

```
./hdfs namenode -format
```
* 切换到hadoop-2.5.2文件夹下启动hdfs

```
sh ./sbin/start-dfs.sh
启动成功后，可通过jps命令查看
hadoop@coldface-laptop:~/hadoop-2.5.2/sbin$ jps
8608 SecondaryNameNode
8407 DataNode
8717 Jps
8301 NameNode
```
此时可以访问Web界面http://localhost:50070来查看Hadoop的信息，如下所示

![图1](/static/images/hadoop1.pic.jpg)

* 启动yarn

```
sh ./sbin/start-yarn.sh
启动成功后，可通过jps命令查看
27021 DataNode  
27191 SecondaryNameNode  
26899 NameNode  
27367 ResourceManager  
27487 NodeManager  
28043 Jps  
```
此时可以通过Web界面来查看NameNode运行状况(http://localhost:8088),如下图所示
![图2](/static/images/hadoop2.pic.jpg)

到此hadoop已经安装成功！

###安装过程遇到问题
####1.提示用户不在sudoers文件中。此时将被被告，解决方案
```
vi /etc/sudoers
在root ALL=(ALL:ALL) ALL下面新增 hadoop ALL=(ALL:ALL) ALL
最终结果如下:
root ALL=(ALL:ALL) ALL
hadoop ALL=(ALL:ALL) ALL
```
####2.ssh:connect to host localhost port 22:Connection refused,解决方案
#####错误原因：
```
1.sshd未安装
2.sshd未启动
3.防火墙
4.重启ssh服务
```
#####解决方案
```
#安装sshd:
sudo apt-get install openssh-server
#启动sshd:
sudo net start sshd
#检查防火墙设置，关闭防火墙
sudo ufw disable
```
#####验证是否成功
```
hadoop@coldface-laptop:/home/coldface$ ps -e|grep ssh
6617 ?        00:00:00 sshd
6689 pts\6    00:00:01 ssh
6690 ?        00:00:00 sshd
6748 ?        00:00:00 sshd
7639 pts/20   00:00:01 ssh
7640 ?        00:00:00 sshd
7694 ?        00:00:00 sshd 
```
####3.启动dfs报错
```
执行 sh ./sbin/start-dfs.sh
错误信息：./sbin/start-dfs.sh:82: /home/hadoop/hadoop-2.5.2/sbin/../libexec/hadoop-config.sh: Syntax error:wordunexpected(expecting")")
原因是：环境变量配置少东西(${HADOOP_HOME}/sbin)
之前配置:
export PATH=${HADOOP_HOME}/bin:$PATH
之后配置:
export PATH=${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH
```



