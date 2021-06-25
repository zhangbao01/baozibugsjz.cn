---
title: Docker
---
## docker运行java
### 安装java镜像
```bash
$ docker pull java
```
### 确认是否安装成功
```bash
$ docker images
```
### 配置Dockerfile
自定义目录  创建 `Dockerfile` 文件
```bash
$ FROM java:8
  MAINTAINER baozi
  ADD script-0.0.1-SNAPSHOT.jar stock_script.jar
  EXPOSE 10000
  ENTRYPOINT ["java","-jar","stock_script.jar"]
```
* `FROM`指定基础镜像，必须是第一，这里指定的java，冒号后面是版本
* `MAINTAINER` 维护者信息
* `ADD`将本地文件添加到镜像中，后面是别名
* `EXPOSE`指定外界交互端口
* `ENTRYPOINT`配置容器，使其可执行化，也可以用`CMD`
### 构建镜像
```bash
$ docker build -t stock_script .
```
* `-t`或`--tag`镜像的名字及标签
* 末尾的`.`表示当前文件夹下的Dockerfile
### 启动镜像
```bash
$  docker run -d --restart=always --name stock_company --network host -p 10000:10000  stock_company 
```
* `-d`表示后台运行
* `--restart`重启的时候自动运行
* `--name`重命名
* `--network`指定网卡，注意：不加会导致多docker服务不能相互通信
* `-p`指定端口映射
### 查看容器运行状态
```bash
$ docker ps
```
### 查看所有容器
```bash
$ docker ps -a
```
### 查看容器日志
先通过 `docker ps`获取容器id
```bash
$ docker logs 容器Id
```
### 停止/启动容器
```bash
$ docker stop 容器Id
$ docker start 容器Id
```
### 删除容器
```bash
$ docker rm 容器Id
```
### 删除镜像
```bash
$ docker rmi 镜像Id
```