# 官网下载安装包

> https://www.mongodb.com/try/download/community

![image-20200909125927838](http://39.96.170.240:81/6aa5d763-233b-4207-b3ce-1f506ebdf8ba.png)

使用 `wget` 命令下载，网速不好，可以直接浏览器下载，再传到服务器

```bash
wget https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/4.4/x86_64/RPMS/mongodb-org-server-4.4.0-1.amzn1.x86_64.rpm
```

下载的文件是 `rpm` 格式的安装包

# 安装

使用 `rpm -i <需要安装的包文件名>` 命令进行安装

```bash
rpm -i mongodb-org-server-4.4.0-1.amzn1.x86_64.rpm
```

没有报错，表示安装成功

# 配置mongoDB

使用默认配置即可，端口号是27017，默认后台启动

配置文件位置在`/etc/mongo.conf`, 如果需要特殊配置，可以修改这个文件

# 启动

```bash
 mongod --config /etc/mongod.conf
```

# 卸载

使用 `rpm -e`

```
rpm -i mongodb-org-server-4.4.0-1.amzn1.x86_64
```



## 错误 

> Job for mongod.service failed because the control process exited with error code. See "systemctl status mongod.service" and "journalctl -xe" for details. 

运行

```bash
systemctl status mongod.service
```

> loaded (/etc/rc.d/init.d/mongod; bad; vendor preset: disabled)









