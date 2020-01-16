### proto文件

```protobuf
syntax = "proto3";
package grpcDemo;

message HelloRequest {
   string name = 1;
}

message HelloReply {
   string message = 1;
}

service gRPC {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
```

### makefile文件

```makefile
PHONY: proto
proto:
	protoc -I ./ ./grpchello.proto --go_out=plugins=grpc:.
```

执行 `make proto` 生成 `grpchello.pb.go` 文件 里面包含定义的消息结构体和服务函数

### go服务端源码

```go
// Package main implements a server for Greeter service.
package main

import (
    "context"
    "log"
    "net"

    "google.golang.org/grpc"

    pb "com.deer/prototest/grpcDemo"
)

const (
    port = ":9007"
)

// server is used to implement helloworld.GreeterServer.
type server struct {
    pb.UnimplementedGRPCServer
}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
    log.Printf("Received: %v", in.GetName())
    return &pb.HelloReply{Message: "Hello " + in.GetName()}, nil
}

func main() {
    lis, err := net.Listen("tcp", port)
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }
    s := grpc.NewServer()
    pb.RegisterGRPCServer(s, &server{})
    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

执行后可以与C#写的客户端通信

### go客户端源码

```go
// Package main implements a client for Greeter service.
package main

import (
    "context"
    "log"
    "os"
    "time"

    "google.golang.org/grpc"

    pb "com.deer/prototest/grpcDemo"
)

const (
    address     = "localhost:9007"
    defaultName = "world"
)

func main() {
    // Set up a connection to the server.
    conn, err := grpc.Dial(address, grpc.WithInsecure(), grpc.WithBlock())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    c := pb.NewGRPCClient(conn)

    // Contact the server and print out its response.
    name := defaultName
    if len(os.Args) > 1 {
        name = os.Args[1]
    }
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()
    r, err := c.SayHello(ctx, &pb.HelloRequest{Name: name})
    if err != nil {
        log.Fatalf("could not greet: %v", err)
    }
    log.Printf("来自: %s", r.GetMessage())
}
```

现在C# 的客户端和go的客户端可以跟go的服务端和C#服务端互相通讯