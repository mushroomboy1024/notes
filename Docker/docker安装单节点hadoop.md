## 前提

- 在linux中已经安装了docker

  ``` shell
  [hadoop@sz-pro-hadoop-srv01 ~]$ sudo docker version
  Client: Docker Engine - Community
   Version:           19.03.14
   API version:       1.40
   Go version:        go1.13.15
   Git commit:        5eb3275d40
   Built:             Tue Dec  1 19:20:42 2020
   OS/Arch:           linux/amd64
   Experimental:      false
  
  Server: Docker Engine - Community
   Engine:
    Version:          19.03.14
    API version:      1.40 (minimum version 1.12)
    Go version:       go1.13.15
    Git commit:       5eb3275d40
    Built:            Tue Dec  1 19:19:17 2020
    OS/Arch:          linux/amd64
    Experimental:     false
   containerd:
    Version:          1.3.9
    GitCommit:        ea765aba0d05254012b0b9e595e995c09186427f
   runc:
    Version:          1.0.0-rc10
    GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
   docker-init:
    Version:          0.18.0
    GitCommit:        fec3683
  ```

+ 官方说明文档：http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html

## 拉取镜像

```shell
sudo docker pull sequenceiq/hadoop-docker:2.7.0
```

## 配置文件

```
docker run -it -p 50070:50070 -p 9000:9000 -p 8088:8088 -p 8040:8040 -p 8042:8042  -p 49707:49707  -p 50010:50010  -p50075:50075  -p 50090:50090 sequenceiq/hadoop-docker:2.7.0 /etc/bootstrap.sh -bash
```

## 执行任务

``` shell
# 1.执行
bash-4.1@/usr/local/hadoop# bash ./bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar wordcount hdfs://51ec096df5e6:9000/input/ hdfs://51ec096df5e6:9000/output/count
# 2.查看结果
bash-4.1@/usr/local/hadoop# bash ./bin/hadoop fs -ls /output/count
Found 2 items
-rw-r--r--   1 root supergroup          0 2021-02-07 01:56 /output/count/_SUCCESS
-rw-r--r--   1 root supergroup         29 2021-02-07 01:56 /output/count/part-r-00000
bash-4.1@/usr/local/hadoop# bash ./bin/hadoop fs -cat /output/count/part-r-00000
C#      1
Flink   4
Hadoop  2
JAVA    1
```

