---
title: Docker Compose
date: 2022-06-11 22:11:23
categories: "Docker"
tags:
	- Docker
---

## 简介
Docker Compose 是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建，启动和停止
## 使用步骤：

1. 利用 Dockerfile 定义运行环境镜像 （需要写好 Dockerfile 文件）
1. 使用 docker-compose.yml 定义组成应用的各服务（启动顺序、关联关系等）
1. 运行 docker-compose up 启动应用

## yml 配置指令

- 指定本 yml 依从的 compose 哪个版本制定的:  `version`
- 指定为构建镜像上下文路径 : `build`
   - 上下文路径: `context`
   - 指定构建镜像的 Dockerfile 文件名 : `dockerfile`
   - 添加构建参数，这是只能在构建过程中访问的环境变量 : `args`
   - 设置构建镜像的标签: `labels`
   - 多层构建，可以指定构建哪一层: `target`
- 端口指定映射 `宿主机:容器` : `ports`
- 数据卷挂载   `宿主机:容器` ：`volumes`
- 指定容器运行镜像 ：`image`
- 添加或删除容器拥有的宿主机的内核功能 :  `cap_add`，`cap_drop`
- 指定自定义容器名称，而不是生成的默认名称:  `container_name`
- 设置依赖关系 :  `depends_on`
## YAML 内容
### 例子一
```yaml
version: '3.3'
services:
  unpub:
    build: ./  # 构建镜像在根目录
    container_name: unpub # 容器名
    restart: always  # 自启动
    ports:
      - 4000:4000  # 端口映射，即 -p 参数，将宿主机的4000端口映射到容器的4000端口
    volumes: 
      - ~/.unpub-packages:/app/unpub-packages # 数据卷挂载，即目录映射，-v 参数，（宿主机目录:容器目录）
    depends_on:
      - mongo  # 依赖与下面的自定义镜像
  mongo: 
    image: mongo:4.2.19  # 依赖镜像:版本
    container_name: unpub_mongo  # 容器名称
    restart: always # 自启动
    volumes: 
      - ~/.unpub_mongo:/data/db # 数据卷挂载
```
### 例子二
```yaml
version: '3'
services: 
  nginx: # 服务名字自行定义
    image: nginx # 指定通过nginx该镜像进行启动，这里可以指定镜像的版本号，如果不指定，默认为latest
    ports: # 端口映射，即 -p 参数，将宿主机的80端口映射到容器的80端口
      - 8090:80
    links: # 指明当前服务可以访问到jenkins这个服务
      - jenkins
    volumes: # 目录映射，即 -v 参数，（宿主机目录:容器目录）
      - ~/data/lxf/nginx/conf.d:/etc/nginx/conf.d
  jenkins: # 服务名字自行定义
    image: jenkins
    expose:  # 暴露指定的端口
      - "8080"
      - "5000"
```
## 参考

- [菜鸟 Docker Compose](https://www.runoob.com/docker/docker-compose.html)
- [Docker - Compose的使用](https://juejin.cn/post/6947562482827264007)
