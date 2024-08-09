Docker 运行参数

docker run 镜像 <在镜像中运行的命令>

```bash
docker run nginx:1.17.4-alpine
```

- -i 允许对容器内的标准输入进行交互
- -t 在新容器内指定一个终端或者伪终端

在终端交互中使用 `exit` 退出终端

- -d 启动一个后台运行的docker容器

  会输出一串字符串，这个字符串叫容器的id

使用 `docker ps` 查看正在后台运行的容器

```bash
runoob@runoob:~$ docker ps
CONTAINER ID        IMAGE                  COMMAND              ...  
5917eac21c36        ubuntu:15.10           "/bin/sh -c 'while t…"    ...
```

CONTAINER ID  ：容器id

IMAGE ：使用的镜像

COMMAND ：启动容器时运行的命令

CREATED：容器的创建时间

STATUS：容器的状态



使用 `docker stop <CONTAINER ID>` 停止容器

