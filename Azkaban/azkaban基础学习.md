# azkaban3.x的基础学习

## 一、azkaban的linux安装

azkaban主要有三个组件：元数据、web服务器、executor执行器

+ 元数据：azkaban使用关系型数据库（默认配置文件中设置了mysql）来管理自己的产生的数据，主要是存储任务信息、定时信息、用户管理等元数据
+ web服务器：azkaban提供一个可视化的web管理页面，可以很直观的看见所有的任务以及对任务的操作，任务到点时发送执行命令到executor上（任务分发）
+ executor执行器：真正执行任务的程序，可以启动多个执行器，任务的执行由web服务器分发时选择的策略决定哪个执行器来执行。

#### 1.下载安装包

**以下操作都是使用的是root用户操作**

```shell
$ wget https://github.com/azkaban/azkaban/archive/3.84.4.tar.gz
$ ll
-rw-r--r-- 1 root root 19282093 Apr 25 15:06 3.84.4.tar.gz
```

#### 2.解压到指定目录/usr/local/下

```shell
$ tar -zxvf ./3.84.4.tar.gz -C /usr/local/
$ cd /usr/local
$ ll
drwxrwxr-x  24 root root 4096 Apr  8  2020 azkaban-3.84.4
$ cd azkaban-3.84.4
$ ll
drwxrwxr-x 3 root root    37 Apr  8  2020 az-core
drwxrwxr-x 4 root root    52 Apr  8  2020 az-crypto
drwxrwxr-x 5 root root    86 Apr  8  2020 az-examples
drwxrwxr-x 3 root root    37 Apr  8  2020 az-exec-util
drwxrwxr-x 3 root root    54 Apr  8  2020 az-flow-trigger-dependency-plugin
drwxrwxr-x 3 root root    53 Apr  8  2020 az-flow-trigger-dependency-type
drwxrwxr-x 3 root root    37 Apr  8  2020 az-hadoop-jobtype-plugin
drwxrwxr-x 3 root root    37 Apr  8  2020 az-hdfs-viewer
-rw-rw-r-- 1 root root 21925 Apr  8  2020 az-intellij-style.xml
drwxrwxr-x 4 root root    49 Apr  8  2020 az-jobsummary
drwxrwxr-x 3 root root    55 Apr  8  2020 azkaban-common
drwxrwxr-x 3 root root    55 Apr  8  2020 azkaban-db				# 元数据sql
drwxrwxr-x 3 root root    55 Apr  8  2020 azkaban-exec-server		# executor执行器
drwxrwxr-x 3 root root    37 Apr  8  2020 azkaban-hadoop-security-plugin
drwxrwxr-x 3 root root    55 Apr  8  2020 azkaban-solo-server
drwxrwxr-x 3 root root    54 Apr  8  2020 azkaban-spi
drwxrwxr-x 3 root root   100 Apr  8  2020 azkaban-web-server		# web服务
drwxrwxr-x 3 root root    37 Apr  8  2020 az-reportal
-rw-rw-r-- 1 root root 10291 Apr  8  2020 build.gradle
drwxrwxr-x 3 root root    37 Apr  8  2020 cached-http-filesystem
-rw-rw-r-- 1 root root  6409 Apr  8  2020 CONTRIBUTING.md
drwxrwxr-x 3 root root  4096 Apr  8  2020 docs
drwxrwxr-x 3 root root    21 Apr  8  2020 gradle
-rw-rw-r-- 1 root root  1488 Apr  8  2020 gradle.properties
-rwxrwxr-x 1 root root  5296 Apr  8  2020 gradlew
-rw-rw-r-- 1 root root  2260 Apr  8  2020 gradlew.bat
-rw-rw-r-- 1 root root 11358 Apr  8  2020 LICENSE
-rw-rw-r-- 1 root root  2359 Apr  8  2020 NOTICE
-rw-rw-r-- 1 root root  2406 Apr  8  2020 README.md
-rw-rw-r-- 1 root root    31 Apr  8  2020 requirements.txt
-rw-rw-r-- 1 root root  1203 Apr  8  2020 settings.gradle
drwxrwxr-x 6 root root   124 Apr  8  2020 test
drwxrwxr-x 2 root root    78 Apr  8  2020 tools
```

#### 3.安装mysql

