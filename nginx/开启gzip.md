##  nginx正确开启gzip的方式

在 server 层级 加 gzip:  

```
    server {
        gzip on  开启gzip

        gzip_min_length 字节多大进行压缩
        
        gzip_buffers 设置缓冲区大小
        
        gzip_comp_level   压缩级别官网建议是6
        
        gzip_types  压缩的类型
        
        gzip_vary 添加到header中一般开起
        
        gzip_disable  在IE6不友好
        
        gzip_proxied 反向代理启用
    }
    /*
     *  例：
    */
    server {
        gzip    on;
        gzip_types  application/javascript text/plain application/x-javascript text/css application/xml text/javascript image/jpeg image/gif image/png;
        #设置压缩缓冲区大小，此处设置为4个8K内存作为压缩结果流缓存
        gzip_buffers 4 8k;
        #压缩级别1-9
        gzip_comp_level 9;
        #给响应头加个vary，告知客户端能否缓存
        gzip_vary on;
    }
```