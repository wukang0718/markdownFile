# 用 golang 创建 RPC 服务端和客户端并完成通信



## 服务端

```go
package main

import (
	"fmt"
	"net"
	"net/rpc"
)

// 定义类对象

type World struct {
}

// Hello 绑定类方法
func (World) Hello(name string, resp *string) error {
	fmt.Println("接收到客户端参数", name)
	*resp = name + " 你好!"
	return nil
}

func main() {
	// 注册RPC函数，绑定对象方法
	err := rpc.RegisterName("hello", World{})
	if err != nil {
		fmt.Println("注册rpc服务失败", err)
		return
	}
	// 设置监听
	listener, err := net.Listen("tcp", ":1234")
	if err != nil {
		fmt.Println("监听失败", err)
		return
	}
	// 建立连接
	conn, err := listener.Accept()
	if err != nil {
		fmt.Println("建立链接失败", err)
		return
	}
	// 绑定服务
	rpc.ServeConn(conn)
	defer func(conn net.Conn) {
		err := conn.Close()
		if err != nil {
			fmt.Println("关闭失败", err)
		}
	}(conn)
}
```

## 客户端

```go
package main

import (
	"fmt"
	"net/rpc"
)

func main() {
	// 1. 用 rpc 链接服务器
	client, err := rpc.Dial("tcp", "127.0.0.1:1234")
	if err != nil {
		fmt.Println("服务器链接失败", err)
		return
	}
	defer func(client *rpc.Client) {
		err := client.Close()
		if err != nil {
			fmt.Println("关闭链接失败", err)
		}
	}(client)
	// 2. 调用远程函数
	var str string
	err = client.Call("hello.Hello", "hello", &str)
	if err != nil {
		fmt.Prinln("接口调用失败", err)
		return
	}
	fmt.Println("接收到服务器返回值", str)
}
```

