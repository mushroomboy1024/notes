## Docker组成原理

![img](https://img2020.cnblogs.com/blog/2030366/202006/2030366-20200630103057155-2072939834.png)

## Docker常用命令图解



![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.4e00.com%2Fblog%2Fimg%2Flinux%2Fdocker%2Fdocker-commands.png&refer=http%3A%2F%2Fwww.4e00.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1615441318&t=ffc90475b045ef57b34d65fb2c765a8d)



## 具名和匿名挂载

```shell
# 匿名挂载
-v 容器内路径!
[root@dolphin /]$ docker run -d -P --name nginx01 -v /ect/nginx nginx

# 查看所有卷的情况
[root@dolphin /]$ docker volume ls
```

所有的docker容器内的卷，没有指定目录的情况下都是在`/var/lib/docker/volumes`下

我们通过具名挂载可以方便的找到我们的一个卷，大多情况下使用的`具名挂载`

```shell
# 如果确定是具名挂载还是匿名挂载，还是指定路径挂载
-v 容器内路径			   # 匿名挂载
-v 卷名:容器内路径			  # 具名挂载
-v /宿主机路径:容器内路径		# 指定路径挂载
```

扩展

```shell
# 通过 -v 容器内路径:ro rw 改变读写权限
ro readonly 	# 只读,容器内该路径的内容只能通过宿主机来操作，容器内部无法操作！
rw readwrite    # 可读可写

[root@dolphin /]$ docker run -d -P --name xxx -v xxx:xxx/xxx:ro xxx
[root@dolphin /]$ docker run -d -P --name xxx -v xxx:xxx/xxx:rw xxx
```



## 初识Dockerfile

### Docker容器和虚拟机级别的区别

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fblobs.gitbook.com%2Fassets%2F-LHpJ36dV3jbyNsfbeuh%2F-LIDyLc1thaN3ot8U-8M%2F-LIDyM86g22dOluxY20d%2Fimage.png%3Falt%3Dmedia%26token%3D3bbabb11-c7fc-4171-a189-3bf77530dcbb&refer=http%3A%2F%2Fblobs.gitbook.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1615441388&t=8125210ff2e1c17e37cc2d0700234d18)



### 数据卷容器



### Dockerfile

Dockerfile是用来构建docker镜像的文件！命令参数脚本！

构建步骤：

1、编写一个Dockerfile文件

2、docker build构建成为一个镜像

3、docker run运行镜像

4、docker push 发布镜像（DockerHub、阿里云镜像仓库...）

### DockerFile构建过程

1、每个保留关键字（指令）都是必须大写字母

2、从上到下顺序执行

3、# 表示注释

4、每一个指令都会创建提交一个新的镜像层，并提交

![img](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=1141557534,2963505983&fm=11&gp=0.jpg)

### Dockerfile常用指令

```shell
FROM			# 基础镜像，一切从这里开始构建
MAINTAINER		# 镜像是谁写的，姓名+邮箱
RUN				# 镜像构建的时候需要运行的命令
ADD				# 步骤，tomcat镜像，这个tomcat压缩包！添加内容
WORKDIR			# 镜像的工作目录
VOLUME			# 挂载的目录
EXPOST			# 保留端口配置
CMD				# 指定这个容器启动的时候运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT		# 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD			# 当构建一个被继承Dockerfile，这个就会运行ONBUILD的指令。触发指令
COPY			# 类似ADD，将我们文件拷贝到镜像中
ENV				# 构建的时候设置环境变量
```

### Dockerfile实战

1、准备镜像文件、压缩包

2、编写dockerfile文件，官方命名`Dockerfile`，build的时候会自动寻找这个文件，就不需要-f指定了

### 小结

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fpic2.zhimg.com%2Fv2-efde71808f83b20ae24d4ed4f2b47171_b.jpg&refer=http%3A%2F%2Fpic2.zhimg.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1615441494&t=8454b301dc39565724425c8b64c3c1b6)



## Docker 网络

### Docker0

Docker0实际上就是172.0.0.1

真实开发中`不建议`使用

### 自定义网络

```shell
# docker启动容器时容忍会给docker0网络 --net bridge
[root@dolphin /]$ docker run -d -P --net bridge xxx
# docker0特点：默认，域名不能访问，  --link可以打通连接

# 自定义网络
[root@dolphin /]$ docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
# 查看现有网络
[root@dolphin /]$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
e01f87fa98cc        bridge              bridge              local
7e5ec4e29f69        host                host                local
e2dad78a117e        mynet               bridge              local
4235737b59e6        none                null                local
6949d7ecd443        somenetwork         bridge              local

# 启动一个容器时，连接自定义网络
[root@dolphin /]$ docker run -it --name centos01 --network mynet centos
[root@dolphin /]$ docker run -it --name centos02 --network mynet centos

# 在自定义网络中各容器之间是可以互相连通的
[root@dolphin /]$ docker exec centos01 ping centos02
PING centos02 (192.168.0.3) 56(84) bytes of data.
64 bytes from centos02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.095 ms
64 bytes from centos02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.032 ms

[root@dolphin /]$ docker exec centos02 ping centos01
PING centos01 (192.168.0.2) 56(84) bytes of data.
64 bytes from centos01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.142 ms
64 bytes from centos01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.045 ms
```

### 网络连通

>两个不同网络的容器进行互相通信，实现原理就是让一个容器绑定多个IP地址（不同网络下的IP地址）

```shell
# 自定义网络
[root@dolphin /]$ docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

# 启动一个mynet网络的容器
[root@dolphin /]$ docker run -it --name mynet-centos --network mynet centos

# 启动一个默认使用docker0（bridge）网络的容器
[root@dolphin /]$ docker run -it --name docker0-centos centos

# 连通前
[root@dolphin /]$ docker exec mynet-centos ping docker0-centos
ping: docker0-centos: Name or service not known
[root@dolphin /]$ docker exec docker0-centos ping mynet-centos
ping: mynet-centos: Name or service not known

# 使用docker0（bridge）网络的容器连通mynet网络，同时mynet网络中的容器也可以连通到docker0（bridge）网络中的容器
[root@dolphin /]$ docker network connect mynet docker0-centos

# 连通后的效果
[root@dolphin /]$ docker exec docker0-centos ping mynet-centos
PING mynet-centos (192.168.0.2) 56(84) bytes of data.
64 bytes from mynet-centos.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.136 ms
64 bytes from mynet-centos.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.045 ms
[root@dolphin /]$ docker exec mynet-centos ping docker0-centos
PING docker0-centos (192.168.0.3) 56(84) bytes of data.
64 bytes from docker0-centos.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.118 ms
64 bytes from docker0-centos.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.110 ms
```

### 实战：搭建Redis集群



## Docker Compose



## Docker Swarm



## Docker Stack



## Docker Secret



## Docker Config

