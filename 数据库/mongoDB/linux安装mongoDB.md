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

```conf
#数据库路径
dbpath=/usr/local/mongodb/data
#日志输出文件路径
logpath=/usr/local/mongodb/logs/mongodb.log
#错误日志采用追加模式
logappend=true
#启用日志文件，默认启用
journal=true
#这个选项可以过滤掉一些无用的日志信息，若需要调试使用请设置为false
quiet=true
#端口号 默认为27017
port=27017
#允许远程访问
bind_ip=0.0.0.0
#开启子进程
fork=true
#开启认证，必选先添加用户，先不开启（不用验证账号密码）
#auth=true
```

# 将mongodb服务加入环境变量

```bash
vi /etc/profile
```

## 在最后一行加入

```
export PATH=/opt/mongodb-linux-x86_64-rhel70-4.4.1/bin:$PATH
```



