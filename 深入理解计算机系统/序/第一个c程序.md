## 安装c环境

### 下载编译器

[MinGW-w64 - for 32 and 64 bit Windows](https://link.zhihu.com/?target=https%3A//sourceforge.net/projects/mingw-w64/files/) 往下稍微翻一下，选最新版本中的`x86_64-posix-seh`

### 配置环境变量

![image-20210505121948023](https://gitee.com/wu_kang0718/image/raw/master//20210505122003111.png)

### 验证

`win+r` 打开 `cmd`,运行 `gcc -v`

## 编写c

vscode 安装插件 `code runner` 、`C/C++` 

### 新建文件 `hello.c`

```c
#include <stdio.h>

int main() 
{
    printf("hello, world\n");
    return 0;
}
```

### 运行

右键选择 `run code`；输出 `hello world` 运行成功

> 如果输出错误消息‘gcc 不是内部或外部命令’

![image-20210505131844016](https://gitee.com/wu_kang0718/image/raw/master//20210505131856649.png)













