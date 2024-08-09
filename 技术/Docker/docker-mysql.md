获取mysql镜像

```bash
docker pull mysql
```

启动mysql

```bash
docker run -itd -e MYSQL_ROOT_PASSWORD="root" -v /Users/aibee/mysql/dockerMysqlConf/:/etc/mysql/conf.d -v /Users/aibee/mysql/dockerMysqlDataSpace/:/var/lib/mysql -p 3306:3306  mysql
```


