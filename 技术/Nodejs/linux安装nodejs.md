# 下载 

官网地址

下载 **Linux Binaries (x64)**

> [https://nodejs.org/en/download/](https://nodejs.org/en/download/)

# 解压

```bash
tar -xvf node-v12.18.3-linux-x64.tar.xz
```

# 检查安装成功

```
cd node-v12.18.3-linux-x64
./bin/node -v
```

# 设置软连接

```bash
ln -s /opt/node-v12.18.3-linux-x64/bin/node /usr/bin/
ln -s /opt/node-v12.18.3-linux-x64/bin/npm /usr/bin/
ln -s /opt/node-v12.18.3-linux-x64/bin/npx /usr/bin/
```

# 检查软连接

```bash
node -v
npm -v
```

