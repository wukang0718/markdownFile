# Express 报错 PayloadTooLargeError: request entity too large

## 原因：

​	请求的实体太大，没有设置 `body-parser` 的`limit` 大小，默认是 **1MB**

## 解决：

​	设置 `body-parser` 的 limit