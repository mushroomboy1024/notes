# Kafka工作中常用指令

> **注意：**`[...]`表示可以写很多个

>  kafka的properties官网文档地址：http://kafka.apache.org/documentation.html#consumerconfigs

###### 查看topic列表

```shell
# 注：zookeeper的默认端口为2181
$ ./kafka-topic.sh --zookeeper <ZOOKEEPER-ADDRESS>:<PORT>,[...] --list
```

###### 生产消息

```shell
# 注：broker的默认端口为9092
$ ./kafka-console-producer.sh --broker-list <BROKER-ADDRESS>:<PORT>,[...] --topic <TOPIC-NAME>
```

###### 消费消息

```shell
# 从头开始消费，注：broker的默认端口为9092
$ ./kafka-console-consumer.sh --bootstrap-server <BROKER-ADDRESS>:<PORT>,[...] --topic <TOPIC-NAME> --from-beginning

$ ./kafka-console-consumer.sh --bootstrap-server sz-pro-hadoop-srv04:6667,sz-pro-hadoop-srv05:6667,sz-pro-hadoop-srv06:6667 --topic DP-FODS-S000-EVT-DEVICE-DATAPOINT-ORDER --from-beginning

./kafka-console-consumer.sh --bootstrap-server 172.28.2.203:6667,172.28.2.239:6667,172.28.2.202:6667 --topic DP-FODS-S000-EVT-DEVICE-DATAPOINT-ORDER --from-beginning

```

###### 查询topic的offset信息

```shell
# 根据时间戳查询topic的offset，注：--time timestamp/-1(latest)/-2(earliest)
$ ./kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list <BROKER-ADDRESS>:<PORT>,[...] --topic <TOPIC-NAME> --time -1

kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list sz-pro-hadoop-srv04:6667,sz-pro-hadoop-srv05:6667,sz-pro-hadoop-srv06:6667, --topic DP-FODS-S000-EVT-DEVICE-DATAPOINT-RPT --time -1

kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 172.28.2.203:6667,172.28.2.239:6667,172.28.2.202:6667, --topic DP-FODS-S000-EVT-DEVICE-DATAPOINT-ORDER --time -1

```

###### 启动kafka的web管理界面

**注：**外部访问端口9001

```shell
cd /usr/local/kafka-manager-1.3.3.23 && nohup bin/kafka-manager -Dconfig.file=conf/application.conf &
```

Window上传文件到Linux之后可能会有文件格式不兼容的问题，主要体现在Window的换行符号是\n\r，而Linux上是\n，导致在Linux上每行尾部多出一个符号\r，以下命令将Linux上的文件内容格式化，vim或vi编辑文件，并在命令模式设置以下命令即可

```shell
:set fileformat=unix
```

Linux查看端口是否被占用

```shell
$ netstat -anop|grep 端口
```

初始化mysql

```shell
mysqldump -h172.28.2.107 -uroot -proot -P13306 --databases meta_data --default-character-set utf8 | mysql -hrm-wz9q9wd5fs79575j8fo.mysql.rds.aliyuncs.com -udata_platform -pU4cdG4rL40aw78MLHguY -P3306
```

