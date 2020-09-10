# 下载 

官网地址

下载 ****Linux Binaries (x64)****

> [https://nodejs.org/en/download/](https://nodejs.org/en/download/)

# 解压

```bash
tar -xvf node-v12.18.3-linux-x64.tar.xz
```

# 编译 安装

```bash
cd node-v12.18.3
./configure
make && make install
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

    gcc 版本太低

    #### 解决：

    重新安装gcc

    ##### 下载地址：[http://ftp.gnu.org/gnu/gcc/](http://ftp.gnu.org/gnu/gcc/)

    ```bash
    wget http://ftp.gnu.org/gnu/gcc/gcc-10.2.0/gcc-10.2.0.tar.gz
    
    tar zxvf gcc-10.2.0.tar.gz
    
    cd gcc-10.2.0
    
    ./contrib/download_prerequisites      # 需要的时间可能有点长
    
    mkdir gcc-build
     
    cd gcc-build/
    
    ../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
    
    make && make install      # 需要的时间相当长了点
    
    gcc -v
    ```

- ### lbzip2: Cannot exec: No such file or directory

    没有安装 `bzip2`

    #### 解决：

    ​	安装 `bzip2`

    ```bash
    yum -y install bzip2
    ```

    

