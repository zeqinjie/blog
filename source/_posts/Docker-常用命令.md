---
title: Docker 常用命令
date: 2022-05-19 22:55:55
categories: "Docker"
tags:
	- Docker
---

## 基础命令
- 帮助信息 `docker help`  
   - 通过 `docker` `COMMAND` `--help` 
   - 例如 `docker` `search` `--help`
- 启动 docker:  `systemctl start docker`
- 关闭 docker:  `systemctl stop docker`
- 重启 docker:  `systemctl restart docker`
- 设置随服务启动而自启动：`systemctl enable docker`
## 镜像常用命令

- 查看镜像列表: `docker images` 或者 `docker image Is`
- 查看镜像明细: `docker inspect` 镜像id
- 拉取镜像: `docker pull` `镜像名:tag`
- 镜像提交历史: `docker history` `镜像名:tag` (`镜像id`)
- 删除镜像: `docker rmi` `镜像名:tag`(`镜像id`)
- 创建镜像tag: `docker tag` `镜像名:tag`  `新镜像名:新tag`
   - 如果镜像的名称和 tag 已经不存在，那么此命令就是新增，否则是修改
- 镜像导出：`docker save` `镜像id``＞` 1.tar 或者 `docker save`  `-o` 1.tar `镜像id`
   - 导出进行的详细信息
   - .tar 后缀可以随意命令 
- 镜像导入: `docker load` `<` 1.tar 或者 `docker load` `-i` 1.tar
   - 如果是同一个镜像，那么则导入失败
- 在组成服务器中搜索镜像仓库: `docker search` `镜像名`


![image](/images/2022/Docker/5.png)

## 容器常用命令

- 查看正在运行的容器：`docker ps`
- 查看所有的容器：`docker ps -a`
- 容器的启动、暫停、恢复、停止：`docker` `start` l `pause` l `unpause` l `stop` `容器id`
- 查看容器内的日志：`docker logs` `容器id`
- 删除容器：`docker rm` `容器id`
- 查看容器详情：`docker inspect` `容器id`
- 容器导出：`docker export` 1.tar `容器id` (只是导出当前信息）
- 容器导入：`docker import` 1.tar `镜像名:tag`（是导入为一个镜像）
- 基于当前容器创建一个镜像：`docker commit`
- 创建并启动一个容器: `docker run`
   - `-d`: 后台运行这个容器，`-i`: 以交互的方式运行，`-t`: 分配为伪终端（通常是`-it`配合使用）
      - 交互形式：伪终端方式
      - 守护进程：后台运行方式
   - 镜像通过 -d 参数以后台程序启动运行，如果容器内部没有可以一直运行的进程，那么容器创建启动后就会立即退出
   - 进入容器两种方式
      - 重新进入：`docker attach` `容器id`
         - 如果有多个终端进入这个容器的话，他们之间是操作同步的
         - `ctrl+p+q` 退出不停止这个容器
         - `exit` 退出并停止这个容器
      - 独立的在容器里面运行一个命令：  `docker exec` `容器id`
```
# 后台运行
docker run -d --name busybox01 busybox/83xxxxx 
# 交互式运行
docker run -it --name busybox01 busybox/83xxxxx 
```

- 创建一个容器:  `docker create` 

![image](/images/2022/Docker/6.png)

## 私有仓库命令

![image](/images/2022/Docker/7.png)

## 参考

- [Docker 从入门到实践](https://vuepress.mirror.docker-practice.com/introduction/)
- [Docker 常用命令](https://blog.csdn.net/leilei1366615/article/details/106267225)
- [Docker - 操作镜像资源](https://juejin.cn/post/6921334022316490765)
- [Docker - 操作容器](https://juejin.cn/post/6923816810157047822)
- [Docker - 私有仓库Registry](https://juejin.cn/post/6923817338479804430)

