# 用golang创建tcp socket

server.go

```go
package main

import (
	"fmt"
	"net"
	"strings"
)

func main() {
	ip := "127.0.0.1"
	port := "8080"
	address := fmt.Sprintf("%s:%s", ip, port)
	// 创建链接
	listener, err := net.Listen("tcp", address)
	if err != nil {
		fmt.Println("server链接出错", err)
		return
	}
	fmt.Println("server链接创建成功")

	for {
		// 监听客户端的链接
		conn, err := listener.Accept()

		fmt.Println("server 监听到请求")
		if err != nil {
			fmt.Println("server 监听失败", err)
			return
		}

		go handler(conn)
	}

}

func handler(conn net.Conn) {
	defer func() {
		_ = conn.Close()
	}()
	buf := make([]byte, 1024)

	// 读取客户端发送的消息
	n, err := conn.Read(buf)

	if err != nil {
		fmt.Println("server 读取出错", err)
		return
	}

	fmt.Println("server 接收到数据", string(buf[:n]))

	str := string(buf[:n])
	// str 转成大写
	str = strings.ToUpper(str)

	// 把大写的数据写回给客户端
	n, err = conn.Write([]byte(str))
	if err != nil {
		fmt.Println("server 写入数据出错", err)
	}
}
```

client.go

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	ip := "127.0.0.1"
	port := "8080"
	address := fmt.Sprintf("%s:%s", ip, port)
	// 创建链接
	conn, err := net.Dial("tcp", address)
	if err != nil {
		fmt.Println("client 创建失败", err)
		return
	}

	fmt.Println("client 创建成功")

	// 给服务器发送消息
	str := "hello world"

	_, err = conn.Write([]byte(str))
	if err != nil {
		fmt.Println("client 写入错误", err)
		return
	}

	// 接收服务器的响应
	buf := make([]byte, 1024)
	n, err := conn.Read(buf)
	if err != nil {
		fmt.Println("client 接收服务器数据出错", err)
		return
	}

	fmt.Println("client 接收到服务器数据", string(buf[:n]))

	err = conn.Close()
	if err != nil {
		fmt.Println("client 关闭出错", err)
		return
	}
}
```

![image-20240418103202227](D:\knowledge-base\技术\go\网络\tcp-scoket\image-20240418103202227.png)

![image-20240418103227989](D:\knowledge-base\技术\go\网络\tcp-scoket\image-20240418103227989.png)