## 1. 需求

　　由于通过 Dockerfile 的镜像无法朔源 Git 的版本，导致无法判断当前版本是否为最新或者指定版本。所以需要在 Docker 里面增加相关版本。

## 2. 原理

* 先通过 GIt 来获取读取的版本号。
  > Git 的版本号可以直接定位到具体的提交记录，换句话来说可以直接下载到对应版本的所有代码。不区分分支。
  >
* 通过 Dockerfile 的 docker build 命令将相关数据传入 Dockerfile 中去。
* Dockerfile 可以通过存放到 `LABEL` 中去，或者 `ENV`

## 3. 范例文件

　　shell 脚本

```shell
gitVersion=$(git rev-parse HEAD)
dateTime=$(date "+%Y年%m月%d日 %H时%M分%S秒")
docker build  \
   --build-arg GIT_PULL_V="$gitVersion"  \
   --build-arg BUILD_DATE="$dateTime"  \
   .
```

　　Dockerfile 脚本

```dockerfile
.....
ARG GIT_PULL_V 
ENV GIT_PULL_V=${GIT_PULL_V:-unknown}

ARG BUILD_DATE 
ENV BUILD_DATE=${BUILD_DATE:-unknown}
....
```

　　构建完成镜像后，可以通过 `docker exec 容器名称或者ID env` 读取到相关的配置。
