---
title: "gRPC初识与基本使用"
date: 2020-90-15T08:37:56+08:00
lastmod: 2020-10-20T01:37:56+08:00
draft: false
tags: ["gRPC","Golang"]
categories: ["Golang"]
author: "钱文军"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---
> 要了解gRPC，就必须先了解RPC协议。随着微服务架构的兴起，RPC的概念越来越火爆，用得越来越多了。

## What
**RPC是什么**

在分布式计算里面，**远程过程调用**（英语：Remote Procedure Call）是一个计算机通信协议，该协议允许运行于一台计算机的程序，调用另一个[地址空间](https://zh.wikipedia.org/wiki/%E5%AE%9A%E5%9D%80%E7%A9%BA%E9%96%93)(通常为一个开放网络的一台计算机)的子程序。而程序员就像调用本地程序一样，无需额外地为这个交互作用编程（无需关注细节）。RPC是一种服务器-客户端（Client/Server）模式，经典实现是一个通过发送请求-接受回应进行信息交互的系统。

![](/post/Go/grpc/grpc.svg)

### gRPC是什么
[gRPC](https://www.oschina.net/p/grpc-framework)是一个高性能、开源和通用的 RPC 框架，面向移动和 HTTP/2 设计。目前提供 C、Java 和 Go 语言版本，分别是：grpc, grpc-java, grpc-go. 其中 C 版本支持 C, C++, Node.js, Python, Ruby, Objective-C, PHP 和 C# 支持.

gRPC 基于 HTTP/2 标准设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等特等特性。

在gRPC里，客户端可以像调用本地方法一样直接调用其他机器上的服务端应用程序的方法，帮助你更容易创建分布式应用程序和服务。与许多RPC系统一样，gRPC是基于定义一个服务，指定一个可以远程调用的带有参数和返回类型的的方法。在服务端程序中实现这个接口并且运行gRPC服务处理客户端调用。在客户端，有一个stub提供和服务端相同的方法。

## Why?
使用`gRPC`， 我们可以一次性的在一个`.proto`文件中定义服务并使用任何支持它的语言去实现客户端和服务端，反过来，它们可以应用在各种场景中，从Google的服务器到你自己的平板电脑—— `gRPC`帮你解决了不同语言及环境间通信的复杂性。使用`protocol` `buffers`还能获得其他好处，包括高效的序列号，简单的`IDL`以及容易进行接口更新。总之一句话，使用gRPC能让我们更容易编写跨语言的分布式代码。

## How
如何在Go语言中使用gRPC？
### 安装gRPC

```
go get -u google.golang.org/grpc
```
安装用于生成gRPC服务代码的协议编译器，最简单的方法是从下面的链接：https://github.com/google/protobuf/releases 下载适合你平台的预编译好的二进制文件（protoc-<version>-<platform>.zip）。

下载完之后，执行下面的步骤：

解压下载好的文件
把protoc二进制文件的路径加到环境变量中

你可以打开终端，执行`protoc --version`验证是否成功，正常情况下会看到类似如下的输出：

```
C:\Users\jalen>protoc --version
libprotoc 3.13.0
```

接下来执行下面的命令安装protoc的Go插件：

```
go get -u github.com/golang/protobuf/protoc-gen-go
```
这个命令执行完后，编译插件`protoc-gen-go`将会安装到`$GOBIN`，默认是`$GOPATH/bin`，它必须在你的`$PATH`中以便协议编译器`protoc`能够找到它。

tips:如果你的系统里面`$GOPATH/bin`目录不在`Path`环境变量中，请手动添加这个环境变量。

### 使用gRPC，gRPC入门示例

`gRPC`是基于`Protocol Buffers`。

`Protocol Buffer`是一种与语言无关，平台无关的可扩展机制，用于序列化结构化数据。使用`Protocol Buffers`可以一次定义结构化的数据，然后可以使用特殊生成的源代码轻松地在各种数据流中使用各种语言编写和读取结构化数据。

关于`Protocol Buffers`的教程可以自行在网上搜索，本文默认读者熟悉`Protocol Buffers`。

#### gRPC的开发步骤

gRPC的开发步骤分三步

1. 编写.proto文件，并通过命令，生成指定语言源代码。
2. 编写服务端代码
3. 编写客户端代码

#### 编写.proto文件

这里假设我们创建了一个名为 `gRPC_Demo` 的Go项目，我们在这个项目下的 `/helloworld/pb/` 目录下添加一个名为 `helloworld.proto` 的文件，并写入如下的内容：

```
syntax = "proto3"; // 版本声明，使用Protocol Buffers v3版本

package pb; // 包名


// 定义一个打招呼服务
service Greeter {
    // SayHello 方法
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// 包含人名的一个请求消息
message HelloRequest {
    string name = 1;
}

// 包含问候语的响应消息
message HelloReply {
    string message = 1;
}
```

然后我们执行下面的命令，这个命令会帮我们生成Go语言服务端程序

```
protoc -I helloworld/ helloworld/pb/helloworld.proto --go_out=plugins=grpc:helloworld
```
在 `helloworld.proto` 同级的目录下，会生成 `helloworld.pb.go`文件

#### 编写Server端Go代码

```go
package main

import (
	"context"
	"fmt"
	"github.com/gRPC_Demo/helloworld/pb"
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
	"net"
)

type server struct{}

//实现pb中的接口
func (s *server) SayHello(c context.Context, r *pb.HelloRequest) (*pb.HelloReply, error) {
	return &pb.HelloReply{Message: "Hello " + r.GetName()}, nil
}

//给server定义方法

func main() {
	// 1.使用net包开启一个tcp服务
	lis, err := net.Listen("tcp", ":8972")
	if err != nil {
		fmt.Printf("Listen tcp:8972 failed,err:%#v\n", err)
		return
	}
	// 2.创建一个grpc服务器
	s := grpc.NewServer()

	// 3.将一个实现了pb.GreeterServer接口的结构体注册到rpc服务器中
	// 这个GreeterServer对应的就是我们在proto文件中定义的打招呼服务

	// 注意传入的第二个参数，是一个实现了grpc.GreeterServer接口的结构体指针
	/**
	type GreeterServer interface {
		// SayHello 方法
		SayHello(context.Context, *HelloRequest) (*HelloReply, error)
	}
	*/
	pb.RegisterGreeterServer(s, &server{})

	// 4.给grpc服务器注册反射服务
	reflection.Register(s)

	// 5.Serve方法在lis上接受传入连接，为每个连接创建一个ServerTransport和server的goroutine。
	// 该goroutine读取gRPC请求，然后调用已注册的处理程序来响应它们。
	err = s.Serve(lis)
	if err != nil {
		fmt.Printf("failed to serve: %v", err)
		return
	}
}

```

将上面的代码保存到 `gRPC_demo/helloworld/server/server.go` 文件中，编译并执行：

```
cd helloworld/server
go build
.\server.exe 
# ./server ## mac或者linux环境
```

#### 编写Client端Go代码

```go
package main

import (
	"context"
	"fmt"
	"github.com/gRPC_Demo/helloworld/pb"
	"google.golang.org/grpc"
)

func main() {
	// 1.连接grpc服务器
	conn, err := grpc.Dial(":8972", grpc.WithInsecure())
	if err != nil {
		fmt.Printf("grpc Dial :8972 port failed,err:%v\n", err)
		return
	}
	defer func() { _ = conn.Close() }()

	// 2.使用这个连接，创建打招呼服务的客户端
	client := pb.NewGreeterClient(conn)

	// 3.使用客户端调用SayHello方法，方法的参数和返回值都和服务端一样，就好像直接调用了服务端函数
	r, err := client.SayHello(context.Background(), &pb.HelloRequest{Name: "Lili"})
	if err != nil {
		fmt.Printf("could not greet: %v", err)
		return
	}
	fmt.Printf("Greeting: %s !\n", r.Message)
}
```

将上面的代码保存到 `gRPC_demo/helloworld/client/client.go` 文件中，编译并执行：

```
cd helloworld/client
go build
.\client.exe 

# ./client ## mac或者linux环境
```

这里可以看到如下的输出：

```
$client>client.exe
Greeting: Hello Lili !
```

此时，我们的目录结构如下：
```
.\gRPC_Demo
├── go.mod
├── go.sum
└── helloworld
    ├── client
    │   ├── client
    │   └── client.go
    │   ├── client.py
    ├── pb
    │   ├── helloworld.pb.go
    │   └── helloworld.proto
    └── server
        ├── server
        └── server.go
```

### 跨语言调用


