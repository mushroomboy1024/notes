# 安装Hadoop

> 准备

```shell
[hadoop@sz-pro-hadoop-srv01 docker-hadoop]$ ll
total 480532
-rw-rw-r-- 1 hadoop hadoop       332 Feb 10 14:30 Dockerfile
-rw-rw-r-- 1 hadoop hadoop 348326890 Aug 24 20:40 hadoop-3.1.4.tar.gz
-rw-rw-r-- 1 hadoop hadoop 143722924 Dec 11 03:11 jdk-8u281-linux-x64.tar.gz
```

> 编写Dockerfile

```shell
[hadoop@sz-pro-hadoop-srv01 docker-hadoop]$ vim Dockerfile
FROM centos:7

ADD jdk-8u281-linux-x64.tar.gz /usr/lib
ENV JAVA_HOME /usr/lib/jdk1.8.0_281
ENV JRE_HOME ${JAVA_HOME}/jre
ENV CLASSPATH .:${JAVA_HOME}/lib:${JRE_HOME}/lib

ADD hadoop-3.1.4.tar.gz /usr/lib
ENV HADOOP_HOME /usr/lib/hadoop-3.1.4
ENV PATH ${JAVA_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH

CMD ["/bin/bash"]
```

>创建docker镜像

```shell
[hadoop@sz-pro-hadoop-srv01 docker-hadoop]$ docker build -t fatri/hadoop3.1.4:1.0 .
```

> 创建容器，以特权模式运行容器。

```shell
[root@25f63801292b /]$ docker run -d -name hadoop3.1.4 --privileged=true fatri/hadoop3.1.4:1.0 /usr/sbin/init
```

> 安装ssh

```shell
[root@25f63801292b /]# yum install openssh openssh-clients openssh-server
```

> 配置ssh

```shell
[root@25f63801292b /]# ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
[root@25f63801292b /]# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
[root@25f63801292b /]# chmod 700 ~/.ssh
[root@25f63801292b /]# chmod 600 ~/.ssh/*
```

>启动ssh

```shell
[root@25f63801292b /]# systemctl start sshd.service
# 开机自启
[root@25f63801292b /]# systemctl enable sshd.service
# 测试ssh
[root@25f63801292b /]# ssh localhost
# 查看有关ssh的程序
[root@25f63801292b /]# rpm -qa | grep ssh
# 查看已经启动的ssh服务
[root@25f63801292b /]# ps -ef | grep ssh
```

>编写hadoop配置文件

```shell
# core-site.xml
[root@25f63801292b hadoop]# vi /usr/lib/hadoop/etc/hadoop/core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>

[root@25f63801292b hadoop]# vi /usr/lib/hadoop/etc/hadoop/hdfs-site.xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.secondary.http.address</name>
        <value>localhost:50090</value>
        <description>
            The secondary namenode http server address and port.
            If the port is 0 then the server will start on a free port.
        </description>
	</property>
</configuration>

# 在hadoop-env.sh中添加环境变量
[root@25f63801292b hadoop]# vi /usr/lib/hadoop/etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/lib/jdk1.8.0_281
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

>格式化hadoop的文件系统

```shell
[root@localhost ~]# /usr/lib/hadoop/bin/hdfs namenode -format
```

> 启动hadoop

```shell
[root@25f63801292b hadoop]# /usr/lib/hadoop/sbin/start-all.sh
# 查看hadoop相关进程
[root@25f63801292b hadoop]# jps
9351 Jps
8232 DataNode
8761 ResourceManager
8475 SecondaryNameNode
8909 NodeManager
8079 NameNode
```





## 安装时遇到的问题

>在Docker中使用centos7镜像创建容器后，在里面使用systemctl启动服务报错

Docker的设计理念是在容器里面不运行后台服务，容器本身就是宿主机上的一个独立的主进程，也可以间接的理解为就是容器里运行服务的应用进程。一个容器的生命周期是围绕这个主进程存在的，所以正确的使用容器方法是将里面的服务运行在前台。

再说到systemd，这个套件已经成为主流Linux发行版（比如CentOS7、Ubuntu14+）默认的服务管理，取代了传统的SystemV风格服务管理。systemd维护系统服务程序，它需要特权去会访问Linux内核。而容器并不是一个完整的操作系统，只有一个文件系统，而且默认启动只是普通用户这样的权限访问Linux内核，也就是没有特权，所以自然就用不了！

因此，请遵守容器设计原则，一个容器里运行一个前台服务！

```shell
# 解决该问题，以特权模式运行容器。
$ docker run -d --privileged=true centos:x /usr/sbin/init
```



> 因为是Docker中单节点安装hadoop，所以在hdfs-site.xml配置文件中配置第二个namenode的连接主机名为本地localhost

```shell
    <property>
        <name>dfs.secondary.http.address</name>
        <value>localhost:50090</value>
        <description>
            The secondary namenode http server address and port.
            If the port is 0 then the server will start on a free port.
        </description>
	</property>
```

