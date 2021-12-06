# Docker 安装

## 1. centos7 rpm 离线安装 docker

　　详情看以下教程，基本都囊括了所有的步骤以及突发情况  
[https://www.jianshu.com/p/764ec08196e0?from=singlemessage](https://www.jianshu.com/p/764ec08196e0?from=singlemessage)

## 2. docker 使用

* 开机自启动

```sh
systemctl enable docker
```

* 镜像备份

```sh
docker save nginx:latest > nginx_1.10.tar
```

* 镜像载入

```sh
docker load < nginx_1.10.tar
```

* tag 镜像

```sh
docker tag nginx-test:latest  68.208.6.61:5000/nginx:latest

# nginx-test:latest 现在的名字
# 68.208.6.61:5000/nginx:latest  修改后的名字
```

* 查看当前机器 image

```sh
docker images
```

* 镜像自动更新

```sh
docker run -d \
    --name watchtower \
    --restart unless-stopped \
    -v /var/run/docker.sock:/var/run/docker.sock \
    192.168.6.6:5000/containrrr/watchtower -c \
 --interval 60 \
    nginx redis
    
# nginx redis 具体的容器（启动后 --name 的名字）名字    
# 192.168.6.6:5000 修改为你的私服地址
# --interval 检测的时间（多少秒和服务器通讯获取新的镜像）
```

* 容器自启动

```sh
docker run --restart=always -p 6379:6379 --name redis -v /home/docker/redis/conf/redis.conf:/etc/redis/redis.conf -v /home/docker/redis/data:/data -d redis:6.0.9 redis-server /etc/redis/redis.conf --appendonly yes

# --restart=always 系统重启或者是docker 重启后会自动启动
# 6379:6379 前面为主机端口，后面是容器端口
```

* 进入容器

```sh
docker exec -it 116fecafd7c2 /bin/bash

# 容器的id 116fecafd7c2
```

* 查看 docker 正在运行的容器

```sh
# 查看正在运行的容器
docker ps

# 查看所有容器，包括已经停止的
docker ps -a
```

* 删除镜像/容器

```sh
# 删除容器，可以是容器每次或者容器ID
docker rm redis

# 删除镜像，如果有同名，需要带上 tag
docker rmi redis:latest
```

* 修改私服地址

　　vim /etc/docker/daemon.json

```json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://wevl6ekf.mirror.aliyuncs.com"
  ],"insecure-registries":["192.168.0.22:5000"]
}
```

　　192.168.0.22  修改为具体的私服地址，修改完成后需要重启 docker

```
systemctl restart docker
```

* docker 日志
  https://www.cnblogs.com/operationhome/p/10907591.html

```
/var/lib/docker/containers/container_id/container_id-json.log
```

* 查看容器中的信息

```
docker inspect 容器名称或者id
```

　　查看具体容器 imageid（使用什么镜像运行的容器）

```
docker inspect -f {{".Image"}} 容器名称或者id 
```
