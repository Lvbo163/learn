[原文地址](https://www.jianshu.com/p/1a3f1c3031b5)

# Golang 序列化之 ProtoBuf
**谢烟客**
<!-- TOC -->

- [Golang 序列化之 ProtoBuf](#golang-序列化之-protobuf)
    - [1. 安装：](#1-安装)
    - [2. 安装 ProtoBuf 相关的 golang 依赖库](#2-安装-protobuf-相关的-golang-依赖库)
    - [3. 使用](#3-使用)
        - [3.1 创建 Demo golang工程](#31-创建-demo-golang工程)
        - [3.2 在 example 包中编写 person.proto](#32-在-example-包中编写-personproto)
        - [3.3 进入 Demo 工程的 example 目录，使用 protoc 编译 person.proto](#33-进入-demo-工程的-example-目录使用-protoc-编译-personproto)
        - [3.4 在 golang 工程中使用 protobuf 进行序列化与反序列化](#34-在-golang-工程中使用-protobuf-进行序列化与反序列化)

<!-- /TOC -->


ProtoBuf： 是一套完整的 IDL（接口描述语言），出自Google，基于 C++ 进行的实现，开发人员可以根据 ProtoBuf 的语言规范生成多种编程语言（Golang、Python、Java 等）的接口代码，本篇只讲述 Golang 的基础操作。据说 ProtoBuf 所生成的二进制文件在存储效率上比 XML 高 3\~10 倍，并且处理性能高 1\~2 个数量级，这也是选择 ProtoBuf 作为序列化方案的一个重要因素之一。

## 1. 安装：
安装 protoc ：Protoc下载地址，可以根据自己的系统下载相应的 protoc，windows 用户统一下载 win32 版本。
配置 protoc 到系统的环境变量中，执行如下命令查看是否安装成功：
```ssh
$ protoc --version
```
如果正常打印 libprotoc 的版本信息就表明 protoc 安装成功

## 2. 安装 ProtoBuf 相关的 golang 依赖库
```ssh
$ go get -u github.com/golang/protobuf/{protoc-gen-go,proto}
```

## 3. 使用

### 3.1 创建 Demo golang工程

![demo工程](https://upload-images.jianshu.io/upload_images/208550-5ab44a941d267dff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/429/format/webp)

### 3.2 在 example 包中编写 person.proto

```
syntax = "proto3";
package example;

message person {    //  aa 会生成 Aa 命名的结构体
    int32 id = 1;
    string name = 2;
}

message all_person {    //  aa_bb 会生成 AaBb 的驼峰命名的结构体
    repeated person Per = 1;
}
```

### 3.3 进入 Demo 工程的 example 目录，使用 protoc 编译 person.proto
```ssh
$ protoc --go_out=. person.proto
```
就会生成 person.pb.go 文件

### 3.4 在 golang 工程中使用 protobuf 进行序列化与反序列化
**main.go:**

```go
package main

import (
    "github.com/golang/protobuf/proto"
    "Demo/example"
    "log"
)

func main() {
    // 为 AllPerson 填充数据
    p1 := example.Person{
        Id:*proto.Int32(1),
        Name:*proto.String("xieyanke"),
    }

    p2 := example.Person{
        Id:2,
        Name:"gopher",
    }

    all_p := example.AllPerson{
        Per:[]*example.Person{&p1, &p2},
    }

    // 对数据进行序列化
    data, err := proto.Marshal(&all_p)  
    if err != nil {
        log.Fatalln("Mashal data error:", err)
    }

    // 对已经序列化的数据进行反序列化
    var target example.AllPerson
    err = proto.Unmarshal(data, &target)
    if err != nil{
        log.Fatalln("UnMashal data error:", err)
    }

    println(target.Per[0].Name) // 打印第一个 person Name 的值进行反序列化验证
}
```
