# 安装Docker Compose

```shell
# 1.下载（官方镜像，太慢）
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# （国内镜像）推荐使用
$ sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 2.赋权
$ sudo chmod +x /usr/local/bin/docker-compose

# 3.创建软连接
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
# 验证
$ docker-compose version
docker-compose version 1.25.5, build 8a1c60f6
docker-py version: 4.1.0
CPython version: 3.7.5
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019

```

## yaml规则

docker-compose.yaml配置文件，v3.x的官网文档地址：https://docs.docker.com/compose/compose-file/compose-file-v3/

```shell
# 3层

# 1.版本
version: '' 

# 2.服务
service: 
	服务1:web
		# 服务配置
		images:
		build:
		network:
		...
	服务2:redis
		...
	...

# 3.其他配置 网络/卷、全局规则
volumes:
networks:
configs:
```



## 小结

1、Docker镜像。run => 容器

2、Dockerfile构建镜像（服务打包）

3、docker-compose启动项目（编排、多个容器/环境）

4、Docker网络





