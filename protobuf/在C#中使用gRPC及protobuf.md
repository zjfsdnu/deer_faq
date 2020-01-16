## 简介

gRPC提供了很多的语言开发包，C#也可以很容易使用。结合使用protobuf及其编译器，可以很容易地生成gRPC的服务stub和proxy

在CSharp中使用gRPC和Protobuf，可以简单地使用Nuget安装grpc和protobuf的支持包

protobuf的编译器使用各种语言的支持插件，可以创建各种语言的Message及序列化操作列，以及grpc服务定义类。这些插件有java、c#、python……。

## 简单示例

此示例创建一个gRPC的服务，已经调用client端。利用了protobuf定义文件和protobuf编译器来自动生成框架。

#### 定义一个库项目

在这个库中，使用protobuf定义传递的Message，已经service的api定义。
如下的`grpchello.pb`这是一个protobuf定义文件。

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

接着需要使用protobuf编译器，编译准备好的protobuf定义文件，为我们生成服务定义和消息定义等各种class。

- 先使用nuget安装grpc和protobuf包：搜索Grpc包和Google.Protobuf依赖包，这2个包在下面的server和client工程中也需要安装
- 这个库因为需要使用protobuf的编译器，还需要安装一个Grpc.Tools的工具包

在安装了Grpc.Tools后，在C:\Users\Administrator\.nuget\packages\grpc.tools\2.26.0\tools\windows_x64\下，包括了protoc.exe以及cs的生成插件grpc_csharp_plugin.exe。然后我们可以使用它们编译protobuf定义文件，生成cs源码了

编写makefile文件（批处理也可 请自行编写）

```makefile
DIR = C:\Users\Administrator\.nuget\packages\grpc.tools\2.26.0\tools\windows_x64
PLUGIN = $(DIR)\grpc_csharp_plugin.exe
PROTOC = $(DIR)\protoc.exe

proto:
	$(PROTOC) --csharp_out=. grpchello.proto --grpc_out=. --plugin=protoc-gen-grpc=$(PLUGIN)
```

然后执行 `make proto`，会生成Grpchello.cs和GrpchelloGrpc.cs源码文件。将源码文件添加到工程中，然后编译好库文件。

- 在Grpchello.cs类，定义了传递的Message
- 在GrpchelloGrpc.cs中，定义了grpc服务stub和proxy的形式

#### 创建grpc的服务端

现在，根据我们定义的grpc服务接口API，我们需要实现服务，为调用的客户端提供服务。

创建一个应用工程，就使用最简单的Console 应用。添加上面的库工程依赖。然后使用Nuget安装Grpc依赖。

**实现服务API**

实现gRPC.gRPCBase类，实现我们定义的方法。

```c#
using Grpc.Core;
using GrpcDemo;
using System;
using System.Threading.Tasks;

namespace grpcConsoleServer
{
    class gRPCImpl : gRPC.gRPCBase
    {
        // 实现SayHello方法
        public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
        {
            Console.WriteLine("Get : " + request.Name);
            return Task.FromResult(new HelloReply { Message = "Hello " + request.Name });
        }

    }
}
```

**在应用中启动服务**

使用`Grpc.Core` 中提供的Server类，指定服务端口，和提供服务的实现类。

```c#
using Grpc.Core;
using GrpcDemo;
using System;

namespace grpcConsoleServer
{
    class Program
    {
        static void Main(string[] args)
        {
            const int Port = 9007;

            Server server = new Server
            {
                Services = { gRPC.BindService(new gRPCImpl()) },
                Ports = { new ServerPort("localhost", Port, ServerCredentials.Insecure) }
            };
            server.Start();

            Console.WriteLine("gRPC server listening on port " + Port);
            Console.WriteLine("任意键退出...");
            Console.ReadKey();

            server.ShutdownAsync().Wait();
        }
    }
}
```
#### 创建客户端

为使用grpc服务，需要一个客户端。简单地创建一个Console App应用。
然后添加Grpc依赖包，和开始的工程依赖库grpcDemo。

```c#
using Grpc.Core;
using GrpcDemo;
using System;

namespace grpcConsoleClient
{
    class Program
    {
        static void Main(string[] args)
        {
            Channel channel = new Channel("127.0.0.1:9007", ChannelCredentials.Insecure);

            var client = new gRPC.gRPCClient(channel);
            for (int i = 0; i < 5; i++)
            {
                var reply = client.SayHello(new HelloRequest { Name = " -- client " + i });
                Console.WriteLine("来自" + reply.Message);
            }

            channel.ShutdownAsync().Wait();
            Console.WriteLine("任意键退出...");
            Console.ReadKey();
        }
    }
}
```

如示例所示，客户端通过一个channel创建一个gRPCClient，在Channel中指定了服务地址和安全方式，client为服务的proxy。调用方式很简单，和普通的API一致。

#### 执行测试

编译完成后，分别到服务端和客户端，启动应用，可以看到调用及结果。

————————————————

版权声明：本文为CSDN博主「whereismatrix」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/whereismatrix/article/details/78477090