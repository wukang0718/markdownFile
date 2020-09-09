# 下载 

官网地址

> https://nodejs.org/en/download/

# 解压

```bash
tar -xzvf node-v12.18.3.tar.gz
```

# 编译

```bash
cd node-v12.18.3
./configure
```

## 报错

- ### failed to autodetect C++ compiler version (CXX=g++)

    没有gcc的依赖

    #### 解决：

    ​	安装gcc

    ```bash
    yum search g++ -y
    ```

- ####  C++ compiler (CXX=g++, 4.8.5) too old, need g++ 6.3.0 or clang++ 8.0.0

    g++ 版本太低

    #### 解决：

    ​	