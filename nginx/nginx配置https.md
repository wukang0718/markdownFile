# Nginx配置https

## 证书购买

[阿里云证书购买地址][https://yundunnext.console.aliyun.com/?spm=5176.22414175.782131.4.643977729P9uOL&p=cas#/overview/cn-hangzhou]

## nginx配置

```nginx
server {
        listen  443 ssl;
        server_name img.wukang0718.com;

        ssl_certificate /etc/nginx/cert/4454239_img.wukang0718.com.pem;
        ssl_certificate_key  /etc/nginx/cert/4454239_img.wukang0718.com.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        #表示使用的加密套件的类型。
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #表示使用的TLS协议的类型。
        ssl_prefer_server_ciphers on;
        location / {
                root /etc/nginx/html/cti;
                index demo.html;
        }
}
```

## 异常问题

- 通过HTTPS无法正常访问您的网站。

  1. 安装证书的Nginx服务器的443端口未开放或被其他工具拦截。

     如果您使用的是阿里云ECS服务器，请前往[ECS管理控制台](https://ecs.console.aliyun.com/)的**安全组**页面，配置开放443端口。关于如何配置安全组，请参见[添加安全组规则](https://help.aliyun.com/document_detail/25471.html#concept-sm5-2wz-xdb)。如果您使用的不是阿里云ECS服务器，请参照对应的服务器安全设置指南，配置开放服务器的443端口。

