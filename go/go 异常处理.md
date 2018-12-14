# [Go语言错误处理](https://tonybai.com/2015/10/30/error-handling-in-go/)

近期闲暇用Go写一个lib，其中涉及到error处理的地方让我琢磨了许久。关于Go错误处理的资料和视频已有许多，Go authors们也在官方Articles和Blog上多次提到过一些Go error handling方面的一些tips和best practice，这里仅仅算是做个收集和小结，尽视野所及，如有不足，欢迎评论中补充。（10月因各种原因，没有耕博，月末来一发，希望未为晚矣 ^_^）

<!-- TOC -->
- [1. 概述](#1-概述)
- [2. 惯用法(idiomatic usage)](#2-惯用法idiomatic-usage)
    - [2.1 基本用法](#21-基本用法)
    - [2.2 注意事项](#22-注意事项)
- [3. 槽点与破解之法](#3-槽点与破解之法)
    - [3.1 checkError style](#31-checkerror-style)
    - [3.2 聚合error handle functions](#32-聚合error-handle-functions)
    - [3.3 将doStuff和error处理绑定](#33-将dostuff和error处理绑定)
- [4. 解调用者之惑](#4-解调用者之惑)
    - [4.1 由errors.New或fmt.Errorf返回的错误变量](#41-由errorsnew或fmterrorf返回的错误变量)
    - [4.2 exported Error变量](#42-exported-error变量)
    - [4.3 定义自己的error接口实现类型](#43-定义自己的error接口实现类型)
- [五、坑(s)](#五坑s)
    - [5.1 Go FAQ：Why is my nil error value not equal to nil?](#51-go-faqwhy-is-my-nil-error-value-not-equal-to-nil)
    - [5.2 switch err.(type)的匹配次序](#52-switch-errtype的匹配次序)
- [六、第三方库](#六第三方库)
<!-- /TOC -->

# 1. 概述
Go是一门simple language，常拿出来鼓吹的就是作为gopher习以为傲的仅仅25个关键字^_^。因此Go的错误处理也一如既往的简单。我们知道C语言错误处理以返 回错误码(errno)为主流，目前企业第一语言Java则用try-catch- finally的处理方式来统一应对错误和异常（开发人员常常因分不清楚到底哪些是错误，哪些是异常而滥用该机制）。Go则继承了C，以返回值为错误处理的主要方式（辅以panic与recover应对runtime异常）。但与C不同的是，在Go的惯用法中，返回值不是整型等常用返回值类型，而是用了一个 error(interface类型)。
```go
type interface error {
    Error() string
}
```
这也体现了Go哲学中的“正交”理念：error context与error类型的分离。无论error context是int、float还是string或是其他，统统用error作为返回值类型即可。
```go
func yourFunction(parametersList) (..., error)
func (Receiver)yourMethod(parametersList) (..., error)
```
在Andrew Gerrand的“Error handling and Go“一文中，这位Go authors之一明确了error context是由error接口实现者supply的。在Go标准库中，Go提供了两种创建一个实现了error interface的类型的变量实例的方法：errors.New和fmt.Errorf：
```go
errors.New("your first error code")
fmt.Errorf("error value is %d\n", errcode)
```
这两个方法实际上返回的是同一个实现了error interface的类型实例，这个unexported类型就是errorString。顾名思义，这个error type仅提供了一个string的context！

//$GOROOT/srcerrors/errors.go
```go
// Copyright 2011 The Go Authors.  All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

// Package errors implements functions to manipulate errors.
package errors

// New returns an error that formats as the given text.
func New(text string) error {
    return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
    s string
}

func (e *errorString) Error() string {
    return e.s
}

```
这两个方法也基本满足了大部分日常学习和开发中代码中的错误处理需求。

# 2. 惯用法(idiomatic usage)
## 2.1 基本用法
就像上面函数或方法定义那样：
```go
func yourFunction(parametersList) (..., error)
func (Receiver)yourMethod(parametersList) (..., error)
```
通常情况，我们将函数或方法定义中的最后一个返回值类型定义为error。使用该函数或方法时，通过如下方式判断错误码：
```go
..., err := yourFunction(...)
if err != nil {
    //error handling
}
```
or
```go
if ..., err := yourFunction(...); err != nil {
    //error handling
}
```
## 2.2 注意事项
- 永远不要忽略(ignore)函数或方法返回的错误码，Check it。（例外：包括标准库在内的Go代码很少去判断fmt.Println or Printf系列函数的返回值）

- error的string context中的内容格式：头母小写，结尾不带标点。因为考虑到error被经常这么用：
```go
... err := errors.New("error example")
fmt.Printf("The returned error is %s.\n", err)
```
- error处理流的缩进样式

prefer
```go
..., err := yourFunction(...)
if err != nil {
    // handle error
}

//go on doing something.
```
rather than:
```go
..., err := yourFunction(...)
if err == nil {
    // do something.
}

// handle error
```
# 3. 槽点与破解之法
Go自诞生那天起就伴随着巨大争议，这也不奇怪，就像娱乐圈，如果没有争议，哪有存在感，刷脸的机会都没有。看来有争议是件好事，没争议的编程语言都已经成为了历史。炒作懂么！这也是很多Gopher的微博、微信、twitter、medium账号喜欢发“Why I do not like Go”类文章的原因吧^_^。

Go的error处理方式就是被诟病的点之一，反方主要论点就是Go的错误处理机制似乎回到了70年代（与C同龄^_^），使得错误处理代码冗长且重复(部分也是由于前面提到的：不要ignore任何一个错误码)，比如一些常见的错误处理代码形式如下：
```go
err := doStuff1()
if err != nil {
    //handle error...
}

err = doStuff2()
if err != nil {
    //handle error...
}

err = doStuff3()
if err != nil {
    //handle error...
}
```
这里不想去反驳这些论点，Go authors之一的Russ Cox对于这种观点进行过驳斥：当初选择返回值这种错误处理机制而不是try-catch这种机制，主要是考虑前者适用于大型软件，后者更适合小程序。当程序变大，try-catch会让错误处理更加冗长繁琐易出错(具体参见go faq)。不过Russ Cox也承认Go的错误处理机制对于开发人员的确有一定的心智负担。

好了，关于这个槽点的叙述点到为止，我们关心的是“如何破解”！Go的错误处理的确冗长，但使用一些tips，还是可以将代码缩减至可以忍受的范围的，这里列举三种：

## 3.1 checkError style
对于一些在error handle时可以选择goroutine exit（注意：如果仅存main goroutine一个goroutine，调用runtime.Goexit会导致program以crash形式退出）或os.Exit的情形，我们可以选择类似常见的checkError方式简化错误处理，例如：
```go
func checkError(err error) {
    if err != nil {
        fmt.Println("Error is ", err)
        os.Exit(-1)
    }
}

func foo() {
    err := doStuff1()
    checkError(err)

    err = doStuff2()
    checkError(err)

    err = doStuff3()
    checkError(err)
}
```
这种方式有些类似于C中用宏(macro)简化错误处理过程代码，只是由于Go不支持宏，使得这种方式的应用范围有限。

## 3.2 聚合error handle functions
有些时候，我们会遇到这样的情况：
```go
err := doStuff1()
if err != nil {
    //handle A
    //handle B
    ... ...
}

err = doStuff2()
if err != nil {
    //handle A
    //handle B
    ... ...
}

err = doStuff3()
if err != nil {
    //handle A
    //handle B
    ... ...
}
```
在每个错误处理过程，处理过程相似，都是handle A、handle B等，我们可以通过Go提供的defer + 闭包的方式，将handle A、handle B…聚合到一个defer匿名helper function中去：
```go
func handleA() {
    fmt.Println("handle A")
}
func handleB() {
    fmt.Println("handle B")
}

func foo() {
    var err error
    defer func() {
        if err != nil {
            handleA()
            handleB()
        }
    }()

    err = doStuff1()
    if err != nil {
        return
    }

    err = doStuff2()
    if err != nil {
        return
    }

    err = doStuff3()
    if err != nil {
        return
    }
}
```
## 3.3 将doStuff和error处理绑定
在Rob Pike的”Errors are values”一文中，Rob Pike told us 标准库中使用了一种简化错误处理代码的trick，bufio的Writer就使用了这个trick：
```go
    b := bufio.NewWriter(fd)
    b.Write(p0[a:b])
    b.Write(p1[c:d])
    b.Write(p2[e:f])
    // and so on
    if b.Flush() != nil {
            return b.Flush()
        }
    }
```
我们看到代码中并没有判断三个b.Write的返回错误值，错误处理放在哪里了呢？我们打开一下$GOROOT/src/
```go
type Writer struct {
    err error
    buf []byte
    n   int
    wr  io.Writer
}

func (b *Writer) Write(p []byte) (nn int, err error) {
    for len(p) > b.Available() && b.err == nil {
        ... ...
    }
    if b.err != nil {
        return nn, b.err
    }
    ......
    return nn, nil
}
```
我们可以看到，错误处理被绑定在Writer.Write的内部了，Writer定义中有一个err作为一个错误状态值，与Writer的实例绑定在了一起，并且在每次Write入口判断是否为!= nil。一旦!=nil，Write其实什么都没做就return了。

以上三种破解之法，各有各的适用场景，同样你也可以看出各有各的不足，没有普适之法。优化go错误处理之法也不会局限在上述三种情况，肯定会有更多的solution，比如代码生成，比如其他还待发掘。

# 4. 解调用者之惑
前面举的例子对于调用者来讲都是较为简单的情况了。但实际编码中，调用者不仅要面对的是：
```go
if err != nil {
    //handle error
}
```
还要面对：
```go
if err 是 ErrXXX
    //handle errorXXX

if err 是 ErrYYY
    //handle errorYYY

if err 是ErrZZZ
    //handle errorZZZ
```

我们分三种情况来说明调用者该如何处理不同类型的error实现：

## 4.1 由errors.New或fmt.Errorf返回的错误变量
如果你调用的函数或方法返回的错误变量是调用errors.New或fmt.Errorf而创建的，由于errorString类型是unexported的，因此我们无法通过“相当判定”或type assertion、type switch来区分不同错误变量的值或类型，唯一的方法就是判断err.String()是否与某个错误context string相等，示意代码如下：
```go
func openFile(name string) error {
    if file not exist {
        return errors.New("file does not exist")
    }

    if have no priviledge {
        return errors.New("no priviledge")
    }
    return nil
}

func main() {
    err := openFile("example.go")
    if err.Error() == "file does not exist" {
        // handle "file does not exist" error
        return
    }

    if err.Error() == "no priviledge" {
        // handle "no priviledge" error
        return
    }
}
```
但这种情况太low了，不建议这么做！一旦遇到类似情况，就要考虑通过下面方法对上述情况进行重构。

## 4.2 exported Error变量
打开$GOROOT/src/os/error.go，你会在文件开始处发现如下代码：
```go
var (
    ErrInvalid    = errors.New("invalid argument")
    ErrPermission = errors.New("permission denied")
    ErrExist      = errors.New("file already exists")
    ErrNotExist   = errors.New("file does not exist")
)
```
这些就是os包export的错误码变量，由于是exported的，我们在调用os包函数返回后判断错误码时可以直接使用等于判定，比如：
```go
err := os.XXX
if err == os.ErrInvalid {
    //handle invalid
}
... ...
```
也可以使用switch case：
```go
switch err := os.XXX {
    case ErrInvalid:
        //handle invalid
    case ErrPermission:
        //handle no permission
    ... ...
}
... ...
```
（至于error类型变量与os.ErrInvalid的可比较性可参考go specs。

一般对于库的设计和实现者而言，在库的设计时就要考虑好export出哪些错误变量。

## 4.3 定义自己的error接口实现类型
如果要提供额外的error context，我们可以定义自己的实现error接口的类型；如果这些类型还是exported的，我们就可以用type assertion or type switch来判断返回的错误码类型并予以对应处理。

比如$GOROOT/src/net/net.go：
```go
type OpError struct {
    Op string
    Net string
    Source Addr
    Addr Addr
    Err error
}

func (e *OpError) Error() string {
    if e == nil {
        return "<nil>"
    }
    s := e.Op
    if e.Net != "" {
        s += " " + e.Net
    }
    if e.Source != nil {
        s += " " + e.Source.String()
    }
    if e.Addr != nil {
        if e.Source != nil {
            s += "->"
        } else {
            s += " "
        }
        s += e.Addr.String()
    }
    s += ": " + e.Err.Error()
    return s
}
```
net.OpError提供了丰富的error Context，不仅如此，它还实现了除Error以外的其他method，比如：Timeout（实现net.timeout interface） 和Temporary（实现net.temporary interface）。这样我们在处理error时，可通过type assertion或type switch将error转换为*net.OpError，并调用到Timeout或Temporary方法来实现一些特殊的判定。
```go
err := net.XXX
if oe, ok := err.(*OpError); ok {
    if oe.Timeout() {
        //handle timeout...
    }
}
```

# 五、坑(s)
每种编程语言都有自己的专属坑(s)，Go虽出身名门，但毕竟年轻，坑也不少，在error处理这块也可以列出几个。

## 5.1 Go FAQ：Why is my nil error value not equal to nil?
```go
type MyError string

func (e *MyError) Error() string {
    return string(*e)
}

var ErrBad = MyError("ErrBad")

func bad() bool {
    return false
}

func returnsError() error {
    var p *MyError = nil
    if bad() {
        p = &ErrBad
    }
    return p // Will always return a non-nil error.
}

func main() {
    err := returnsError()
    if err != nil {
        fmt.Println("return non-nil error")
        return
    }
    fmt.Println("return nil")
}
```
上面的输出结果是”return non-nil error”，也就是说returnsError返回后，err != nil。err是一个interface类型变量，其underlying有两部分组成：类型和值。只有这两部分都为nil时，err才为nil。但returnsError返回时将一个值为nil，但类型为*MyError的变量赋值为err，这样err就不为nil。解决方法：
```go
func returnsError() error {
    var p *MyError = nil
    if bad() {
        p = &ErrBad
        return p
    }
    return nil
}
```
## 5.2 switch err.(type)的匹配次序
试想一下下面代码的输出结果：
```go
type MyError string

func (e MyError) Error() string {
    return string(e)
}

func Foo() error {
    return MyError("foo error")
}

func main() {
    err := Foo()
    switch e := err.(type) {
    default:
        fmt.Println("default")
    case error:
        fmt.Println("found an error:", e)
    case MyError:
        fmt.Println("found MyError:", e)
    }
    return

}
```
你可能会以为会输出：”found MyError: foo error”，但实际输出却是：”found an error: foo error”，也就是说e先匹配到了error！如果我们调换一下次序呢：
```go
... ...
func main() {
    err := Foo()
    switch e := err.(type) {
    default:
        fmt.Println("default")
    case MyError:
        fmt.Println("found MyError:", e)
    case error:
        fmt.Println("found an error:", e)
    }
    return
}
```
这回输出结果变成了：“found MyError: foo error”。

也许你会认为这不全是错误处理的坑，和switch case的匹配顺序有关，但不可否认的是有些人会这么去写代码，一旦这么写，坑就踩到了。因此对于通过switch case来判定error type的情况，将error这个“通用”类型放在后面或去掉。

# 六、第三方库
如果觉得go内置的错误机制不能很好的满足你的需求，本着“do not reinvent the wheel”的精神，建议使用一些第三方库来满足，比如：juju/errors。这里就不赘述了。

© 2015, bigwhite. 版权所有.
