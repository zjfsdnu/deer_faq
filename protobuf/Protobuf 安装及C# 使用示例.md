## 01| 简介

Protobuf（Protocol Buffers），是 Google 开发的一种跨语言、跨平台的可扩展机制，用于序列化结构化数据。

与 XML 和 JSON 格式相比，protobuf 更小、更快、更便捷。protobuf 目前支持 C++、Java、Python、Objective-C，如果使用 proto3，还支持 C#、Ruby、Go、PHP、JavaScript 等语言。

官网地址：[developers.google.cn/protocol-bu…](https://developers.google.cn/protocol-buffers/)

GitHub 地址：[github.com/protocolbuf…](https://github.com/protocolbuffers/protobuf)

**优点：**

- 性能好
- 跨语言

**缺点：**

- 二进制格式可读性差：为了提高性能，protobuf 采用了二进制格式进行编码，这直接导致了可读性差。
- 缺乏自描述：XML 是自描述的，而 protobuf 不是，不配合定义的结构体是看不出来什么作用的。

## 02| 安装

### 2.1 Windows 下安装

下载地址：[github.com/protocolbuf…](https://github.com/protocolbuffers/protobuf/releases)

下载 protoc-3.11.2-win64.zip，这个是编译后的压缩包，相当于绿色版，解压后，将其下的 bin 目录添加到环境变量就可以了，省去了安装的麻烦。

![img](https://user-gold-cdn.xitu.io/2019/9/10/16d18e8d82e2605d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



然后打开命令提示符，输入命令：

```shell
protoc --version
```

成功显示版本号，则表示安装成功。如下图：

![protobuf 安装（1）.png](https://user-gold-cdn.xitu.io/2019/9/10/16d18e8d8328ef0a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 03| 简单使用

### 3.1 编译

使用 protobuf 首先需要定义 .proto 文件，先来看一个简单的例子。

定义 Person.proto 文件，内容如下：

```protobuf
syntax = "proto3";
package Test;

message Person {
  string Name = 1;
  int32 Age = 2;
  bool Marriage = 3;
}
```

- `syntax = "proto3";` 指定正在使用 proto3 语法，否则 protobuf 将默认使用的是 proto2。
- `package Test;` 指定命名空间（C# 中）。
- `message` 是关键字，定义结构化数据。
- 等号后面的数字是字段唯一编号（**注意不是字段的值**），用于二进制格式消息中标识字段。

**protoc 是 protobuf 自带的编译器**，可以将 .proto 文件编译成 java、python、go、C# 等多种语言的代码，直接引用。

**编译命令：**

```shell
protoc -I=E:\GL\Test2017 --python_out=E:\GL\Test2017 Person.proto
```

**编译命令说明：**

- -I 表示源文件（.proto 文件）所在文件夹路径
- --python_out 表示目标语言为 python，且指定生成的 .py 文件存放目录。相应的，C# 为 `csharp_out`
- Person.proto 为源文件文件名，如果有多个，空格隔开

### 3.2 C# 示例

C# 下的 Protobuf 有 3 个版本：

- Google.ProtoBuf：Google官方版本，[github.com/google/prot…](https://github.com/google/protobuf/tree/master/csharp)
- protobuf-net：.net 社区版本，由 .net 社区爱好者开发，[github.com/mgravell/pr…](https://github.com/mgravell/protobuf-net)
- Google.ProtocolBuffers：据说是由谷歌的 .net 员工在官方版本还未出来的时候开发的，[github.com/jskeet/prot…](https://github.com/jskeet/protobuf-csharp-port)

这里我们介绍谷歌官方版本。

在 VS 中，通过 NuGet 安装 'google.protobuf' 包。

```c#
using Google.Protobuf;
using System;
using Test;

namespace Protobuf
{
    class Program
    {
        static void Main(string[] args)
        {
            Person person = new Person();
            person.Name = "张三";
            person.Age = 20;
            person.Marriage = true;

            // 序列化
            byte[] buffer = person.ToByteArray();

            foreach (byte b in buffer)
            {
                Console.Write(b.ToString("X2") + " ");
            }
            Console.WriteLine();

            // 反序列化
            Person p = Person.Parser.ParseFrom(buffer);

            Console.WriteLine(string.Format("Name: {0}, Age: {1}, Marriage: {2}", p.Name, p.Age, p.Marriage));

            Console.Read();
        }
    }
}
```

**输出：**

```
0A 06 E5 BC A0 E4 B8 89 10 14 18 01
Name: 张三, Age: 20, Marriage: True
```


作者：丹枫无迹
链接：https://juejin.im/post/5d770450f265da03e921f3f8
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。