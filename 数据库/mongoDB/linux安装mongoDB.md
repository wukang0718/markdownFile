# 官网下载安装包

> https://www.mongodb.com/try/download/community

![image-20200911125935964](http://39.96.170.240:81/ed4b5e7a-33e9-4cbc-a6e2-1d78d11cee31.png)

下载的文件是 `tgz`格式的安装包

# 解压

```bash
tar -zxvf mongodb-linux-x86_64-rhel70-4.4.1.tgz
cd mongodb-linux-x86_64-rhel70-4.4.1
```

# 创建mongodb数据存储文件和日志文件

```bash
mkdir -p /usr/local/mongodb/db
mkdir /usr/local/mongodb/logs
touch /usr/local/mongodb/logs/mongodb.log
```

# 创建mongodb配置文件

```bash
touch mongodb.conf
vi mongodb.conf
```

# 编辑文件

```

```









