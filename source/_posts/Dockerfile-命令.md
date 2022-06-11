---
title: Dockerfile 命令
date: 2022-06-11 22:11:08
categories: "Docker"
tags:
	- Docker
---

## 常用命令
- 定义构建时需要的参数:  `ARG` `常量=值`
- 定义镜像时的基础镜像:  `FROM``镜像名:tag`
```
ARG version=1.0.0
FROM golang:${version}
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2127799/1653576675893-4fac00a0-1126-471c-b21f-45e0b24676be.png#clientId=u1d07a874-6bbf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=332&id=u2a24a3c6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=788&originWidth=1038&originalType=binary&ratio=1&rotation=0&showTitle=false&size=94067&status=done&style=none&taskId=ub35f7f37-104e-4900-9a35-7289df464d2&title=&width=437)

> 共用镜像

- 定义进行的标签: `LABEL AUTHOR=zzq`
- 声明暴露的端口: `EXPOSE`
   - 给 dockerfile文件维护者提供信息，在容器启动的时候使用 -P 命令可以可宿主机的端口进行映射
   - 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射
   - 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口
```
EXPOSE 8080
```

- 定义环境变量: `ENV`   在运行的容器中会存在 `ENV USERNAE=zzq`
- 容器启动时执行的命令： `ENTRYPOINT`   
   - 注意存在多个命令时只有最后一个生效
```
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```

- 定义匿名数据卷,在启动容器时忘记挂载数据卷，会自动挂载到匿名卷: `VOLUME`
   - 避免重要的数据，因容器重启而丢失，这是非常致命的
   - 避免容器不断变大。
```
VOLUME ["/root""/home"] 或者 VOLUME /root /home
```

- 指定容器启动后使用的用户: `USER`
- 指定工作目录:`WORKDIR` 
   - `WORKDIR`设置 `RUN、CMD、ENTRYPOIN`
   - 用 `WORKDIR` 指定的工作目录，会在构建镜像的每一层中都存在，通过 `WORKDIR` 创建的目录才会一直存在
- 延迟构建命令的执行: `ONBUILD` 
   - 当基于此镜像构建新的镜像时执行的命令 `ONBUILD RUN  ["echo","hello"]` 
- 给送给容器退出的信号: `STOPSIGNAL`
- 容器健康检查: `HEALTHCHECK` 
   - `healthcheck none` 禁止容器健康检查
   - `healthcheck --interval=3 --timeout=1 --retries=1 curl --fail [http://localhost:8080/ping](http://localhost:8080/ping) || exit 1`
- 用于执行后面跟着的命令行命令：`RUN`
   - shell 格式
```dockerfile
RUN yum install httpd && yum install ftp
```

   - exec 格式
```dockerfile
RUN ["./test.php", "dev", "offline"] # 等价于 RUN ./test.php dev offline
```

- 类似于 `RUN` 指令，用于运行程序:   `CMD`
   - CMD 在docker run 时运行。
   - RUN 是在 docker build。
   - 注意 cmd 执行命令，dockerfile 中只有最后一个cmd命令生效，在启动容器时如果指定了命令，dockerfile中的cmd也会失效
- 添加文件到容器指定目录: `ADD`
   - 文件可以是宿主机上下文目录中的、可以 url 的、也可以使压缩包 (会自动解压）
- 拷贝宿主机上下文目录中的文件到容器中: `COPY`
   - 跟ADD类似，但不具备自动下载或解压的功能
- 用于使用 Dockerfile 创建镜像: `docker build`

## 参考

- [Dockerfile 视频教程](https://www.bilibili.com/video/BV1aq4y1G7QB?p=17&spm_id_from=pageDriver)
   - [相关资料](https://github.com/pingwazi0101/dockerstudy)
- [菜鸟 Dockerfile 命令](https://www.runoob.com/docker/docker-dockerfile.html)
- [Docker - Dockerfile的使用](https://juejin.cn/post/6923818420132085774)
- [docker-compose](https://www.runoob.com/docker/docker-compose.html)
- [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)

