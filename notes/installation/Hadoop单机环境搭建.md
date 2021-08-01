# Hadoop单机版环境搭建

<nav>
<a href="#一前置条件">一、前置条件</a><br/>
<a href="#二配置-SSH-免密登录">二、配置 SSH 免密登录</a><br/>
<a href="#三HadoopHDFS环境搭建">三、Hadoop(HDFS)环境搭建</a><br/>
<a href="#四HadoopYARN环境搭建">四、Hadoop(YARN)环境搭建</a><br/>
</nav>




## 一、前置条件

Hadoop 的运行依赖 JDK，需要预先安装，安装步骤见：

+ [Linux 下 JDK 的安装](https://github.com/heibaiying/BigData-Notes/blob/master/notes/installation/Linux下JDK安装.md)



## 二、配置免密登录

Hadoop 组件之间需要基于 SSH 进行通讯。

#### 2.1 配置映射

配置 ip 地址和主机名映射：

```shell
vim /etc/hosts
# 文件末尾增加
192.168.43.202  hadoop001
```

### 2.2  生成公私钥

zhouad增加
如果没有安装ssh服务，可以先安装
sudo apt-get install openssh-server

执行下面命令行生成公匙和私匙：

```
ssh-keygen -t rsa
```

### 3.3 授权

进入 `~/.ssh` 目录下，查看生成的公匙和私匙，并将公匙写入到授权文件：

```shell
[root@@hadoop001 sbin]#  cd ~/.ssh
[root@@hadoop001 .ssh]# ll
-rw-------. 1 root root 1675 3 月  15 09:48 id_rsa
-rw-r--r--. 1 root root  388 3 月  15 09:48 id_rsa.pub
```

```shell
# 写入公匙到授权文件
[root@hadoop001 .ssh]# cat id_rsa.pub >> authorized_keys
[root@hadoop001 .ssh]# chmod 600 authorized_keys
```
zhouad 增加
上面内容可以终结为，如下命令
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys


## 三、Hadoop(HDFS)环境搭建



### 3.1 下载并解压

下载 Hadoop 安装包，这里我下载的是 CDH 版本的，下载地址为：http://archive.cloudera.com/cdh5/cdh/5/

备注： 我是https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz下载的最新版
```shell
# 解压
tar -zvxf hadoop-2.6.0-cdh5.15.2.tar.gz 
```



### 3.2 配置环境变量

```shell
# vi /etc/profile
```

配置环境变量：

```
export HADOOP_HOME=/usr/hadoop/hadoop-3.3.1
export  PATH=${HADOOP_HOME}/bin:$PATH
```

执行 `source` 命令，使得配置的环境变量立即生效：

```shell
# source /etc/profile
```



### 3.3 修改Hadoop配置

进入 `${HADOOP_HOME}/etc/hadoop/ ` 目录下，修改以下配置：

#### 1. hadoop-env.sh

```shell
# JDK安装路径
export  JAVA_HOME=/usr/java/jdk1.8.0_201/
```

#### 2. core-site.xml

```xml
<configuration>
    <property>
        <!--指定 namenode 的 hdfs 协议文件系统的通信地址-->
        <name>fs.defaultFS</name>
        <!--hadoop001指机器名字-->
        <value>hdfs://hadoop001:8020</value>
    </property>
    <property>
        <!--指定 hadoop 存储临时文件的目录-->
        <name>hadoop.tmp.dir</name>
        <value>/usr/hadoop/hadoop-3.3.1/data</value>
    </property>
</configuration>
```

#### 3. hdfs-site.xml

指定副本系数和临时文件存储位置：

```xml
<configuration>
    <property>
        <!--由于我们这里搭建是单机版本，所以指定 dfs 的副本系数为 1-->
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

#### 4. slaves

配置所有从属节点的主机名或 IP 地址，由于是单机版本，所以指定本机即可：

```shell
hadoop001
```



### 3.4 关闭防火墙

不关闭防火墙可能导致无法访问 Hadoop 的 Web UI 界面：

```shell
# 查看防火墙状态
sudo firewall-cmd --state
# 关闭防火墙:
sudo systemctl stop firewalld.service
```



### 3.5 初始化

第一次启动 Hadoop 时需要进行初始化，进入 `${HADOOP_HOME}/bin/` 目录下，执行以下命令：

```shell
[root@hadoop001 bin]# ./hdfs namenode -format
```


### 3.6 启动HDFS

进入 `${HADOOP_HOME}/sbin/` 目录下，启动 HDFS：

```shell
[root@hadoop001 sbin]# ./start-dfs.sh
```
```shell
[root@nna hadoop-3.2.0]# start-dfs.sh
Starting namenodes on [nna nns]
ERROR: Attempting to operate on hdfs namenode as root
ERROR: but there is no HDFS_NAMENODE_USER defined. Aborting operation.
Starting datanodes
ERROR: Attempting to operate on hdfs datanode as root
ERROR: but there is no HDFS_DATANODE_USER defined. Aborting operation.
Starting journal nodes [dn1 dn3 dn2]
ERROR: Attempting to operate on hdfs journalnode as root
ERROR: but there is no HDFS_JOURNALNODE_USER defined. Aborting operation.
Starting ZK Failover Controllers on NN hosts [nna nns]
ERROR: Attempting to operate on hdfs zkfc as root
ERROR: but there is no HDFS_ZKFC_USER defined. Aborting operation.
```
如果启动报错
在Hadoop安装目录下找到sbin文件夹

在里面修改四个文件

1、对于start-dfs.sh和stop-dfs.sh文件，添加下列参数：

#!/usr/bin/env bash
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root

2、对于start-yarn.sh和stop-yarn.sh文件，添加下列参数：

#!/usr/bin/env bash
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
重新开始start...就可以。


### 3.7 验证是否启动成功

方式一：执行 `jps` 查看 `NameNode` 和 `DataNode` 服务是否已经启动：

```shell
[root@hadoop001 hadoop-2.6.0-cdh5.15.2]# jps
9137 DataNode
9026 NameNode
9390 SecondaryNameNode
```
```shell
root@Linux-ANDONG:/usr/hadoop/hadoop-3.3.1/sbin# jps
11090 NameNode
11781 Jps
11256 DataNode
11530 SecondaryNameNode
```


方式二：查看 Web UI 界面，端口为 `50070`（3.0以上版本的端口是9870）：

<div align="center"> <img width="700px" src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hadoop安装验证.png"/> </div>


## 四、Hadoop(YARN)环境搭建

### 4.1 修改配置

进入 `${HADOOP_HOME}/etc/hadoop/ ` 目录下，修改以下配置：

#### 1. mapred-site.xml

```shell
# 如果没有mapred-site.xml，则拷贝一份样例文件后再修改
cp mapred-site.xml.template mapred-site.xml
```

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

#### 2. yarn-site.xml

```xml
<configuration>
    <property>
        <!--配置 NodeManager 上运行的附属服务。需要配置成 mapreduce_shuffle 后才可以在 Yarn 上运行 MapReduce 程序。-->
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```



### 4.2 启动服务

进入 `${HADOOP_HOME}/sbin/` 目录下，启动 YARN：

```shell
./start-yarn.sh
```



#### 4.3 验证是否启动成功

方式一：执行 `jps` 命令查看 `NodeManager` 和 `ResourceManager` 服务是否已经启动：

```shell
[root@hadoop001 hadoop-2.6.0-cdh5.15.2]# jps
9137 DataNode
9026 NameNode
12294 NodeManager
12185 ResourceManager
9390 SecondaryNameNode
```

方式二：查看 Web UI 界面，端口号为 `8088`：

<div align="center"> <img width="700px" src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/hadoop-yarn安装验证.png"/> </div>


<div align="center"> <img  src="https://gitee.com/heibaiying/BigData-Notes/raw/master/pictures/weixin-desc.png"/> </div>
