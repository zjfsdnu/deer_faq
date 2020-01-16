## 配置环境

1. 下载最新版protoc：https://github.com/protocolbuffers/protobuf/releases 

   比如 [protoc-3.11.2-win64.zip](https://github.com/protocolbuffers/protobuf/releases/download/v3.11.2/protoc-3.11.2-win64.zip) 解压后把protoc.exe 放到 $GOPATH/bin文件夹下

2. 安装 golang 插件

   `go get -u github.com/golang/protobuf/protoc-gen-go`

   默认也会安装到$GOPATH/bin文件夹下（$GOPATH/bin需要添加到PATH环境变量）

3. 在项目根目录下执行 生成 `.pb.go` 文件

   ```protobuf
   syntax = "proto3";
   package tutorial;
   
   message Person {
       string name = 1;
       int32 id = 2; // Unique ID number for this person.
       string email = 3;
   
       enum PhoneType {
           MOBILE = 0;
           HOME = 1;
           WORK = 2;
       }
   
       message PhoneNumber {
           string number = 1;
           PhoneType type = 2;
       }
   
       repeated PhoneNumber phones = 4;
   }
   
   // Our address book file is just one of these.
   message AddressBook {
       repeated Person people = 1;
   }
   ```

   ```shell
   protoc -I=$SRC_DIR --go_out=$DST_DIR $SRC_DIR/addressbook.proto
   protoc -I=$SRC_DIR --csharp_out=$DST_DIR $SRC_DIR/addressbook.proto
   ```

   $SRC_DIR 为源文件路径

4. 可以在go项目里直接调用生成的`.pb.go` 文件

```go
book := &pb.AddressBook{}
// ...

// Write the new address book back to disk.
out, err := proto.Marshal(book)
if err != nil {
        log.Fatalln("Failed to encode address book:", err)
}
if err := ioutil.WriteFile(fname, out, 0644); err != nil {
        log.Fatalln("Failed to write address book:", err)
}
```

```go
// Read the existing address book.
in, err := ioutil.ReadFile(fname)
if err != nil {
        log.Fatalln("Error reading file:", err)
}
book := &pb.AddressBook{}
if err := proto.Unmarshal(in, book); err != nil {
        log.Fatalln("Failed to parse address book:", err)
}
```



### makefile配置参考

```makefile

.PHONY: proto clean go go gotest

proto:
	protoc --go_out=simple simple.proto
	protoc --go_out=tutorial addressbook.proto

go:     add_person_go     list_people_go
gotest: add_person_gotest list_people_gotest

protoc_middleman_go: addressbook.proto
	mkdir -p tutorial # make directory for go package
	protoc --go_out=tutorial addressbook.proto
	@touch protoc_middleman_go

add_person_go: add_person.go protoc_middleman_go
	go build -o add_person_go add_person.go

add_person_gotest: add_person_test.go add_person_go
	go test add_person.go add_person_test.go

list_people_go: list_people.go protoc_middleman_go
	go build -o list_people_go list_people.go

list_people_gotest: list_people.go list_people_go
	go test list_people.go list_people_test.go

clean:
	rm -f protoc_middleman_go tutorial/*.pb.go add_person_go list_people_go
	find . -regex '.*\.out\|.*\.exe\|.*\.i\|.*\.s\|.*\.o' | xargs rm -f

```



