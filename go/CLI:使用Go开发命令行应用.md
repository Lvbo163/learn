# [CLI:使用Go开发命令行应用](http://www.cnblogs.com/hitandrew/p/5802521.html?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com)
**21 Nov 2014 11:34am, by Cory Lanou**

[原文地址](https://thenewstack.io/cli-command-line-programming-with-go/)
[译文地址](http://www.cnblogs.com/hitandrew/p/5802521.html?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com)


<!-- TOC -->
- [Arguments](#arguments)
- [判断传入程序的参数数量](#判断传入程序的参数数量)
- [遍历参数](#遍历参数)
- [Flag 包](#flag-包)
- [flag.Args](#flagargs)
- [无效的flag参数](#无效的flag参数)
- [flag.Usage](#flagusage)
- [获取输入](#获取输入)
    - [fmt.Scanf](#fmtscanf)
    - [bufio.Scanner](#bufioscanner)
- [一个基本的cat程序](#一个基本的cat程序)
- [帮助](#帮助)
- [总结](#总结)
<!-- /TOC -->

CLI或者“command line interface”是用户在命令行下交互的程序。由于通过将程序编译到一个静态文件中来减少依赖，一次Go特别适合开发CLI程序。如果你编写过安装时需要各种依赖的CLI程序你就知道这个是有多重要了。

在这篇博客中我们将介绍使用Go开发CLI的基本知识。

# Arguments
大多数CLI程序都需要输入一些参数。Go 语言将这些参数以字符串slice处理。
```go
var Args []string
```
查找当前应用的名字。
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Program Name is always the first (implicit) argument
    cmd := os.Args[0]

    fmt.Printf("Program Name: %s\n", cmd)
}
```
这个应用再code/example1下，你可以用一下命令编译运行：
```sh
go build
./example1
```
输出的结果是：
```sh
Program Name: ./example1
```
# 判断传入程序的参数数量
为了确定有多少参数传入，可以计算所有参数的长度减1(记住，第一个参数总是程序的名字)。或者可以直接从os.Args[1:]来判断他的长度。
```sh
package main

import (
    "fmt"
    "os"
)

func main() {
    argCount := len(os.Args[1:])
    fmt.Printf("Total Arguments (excluding program name): %d\n", argCount)
}
```
运行./example2 得到的结果将是0。运行./example2 -foo=bar 得到的记过将是1。

# 遍历参数
下面是一个很快速的遍历参数的例子。
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    for i, a := range os.Args[1:] {
        fmt.Printf("Argument %d is %s\n", i+1, a)
    }

}
```

```sh
Running the program with ./example3 -local u=admin --help results in:
Argument 1 is -local
Argument 2 is u=admin
Argument 3 is --help
```
# Flag 包
目前为止我们已经知道如何在一个程序中查找参数的基本的方法。在这个级别查询他们并且将他们赋值给我们的程序是很麻烦的。所有就有了Flag包。
```go
package main

import (
    "flag"
    "fmt"
)

func main() {
    var port int
    flag.IntVar(&port, "p", 8000, "specify port to use.  defaults to 8000.")
    flag.Parse()

    fmt.Printf("port = %d", port)
}
```
我们首先做的是设置一个int类型的默认值是8000，并且有文字提示的标识。

为了让flag包对设置的变量赋值，需要是用flag.Parse()方法。

不加参数的运行这个程序得到的结果是port = 8000，因为我们明确的指定了如果没有参数传递给port，那么就采用默认的8000.

运行./example4 -p=9000 结果是 port = 9000

同时flag提供了 “program useage”的输出。如果我们运行 ./example4 -help 我们会得到：
```sh
Usage of ./example4:
-p=8000: specify port to use.  defaults to 8000.
```
# flag.Args
很多CLI程序同时包含有标识和没有标识的参数。flag.Args() 将会直接返回哪些没有标识的参数。
```go 
package main

import (
    "flag"
    "fmt"
)

func main() {
    var port int
    flag.IntVar(&port, "p", 8000, "specify port to use.  defaults to 8000.")
    flag.Parse()

    fmt.Printf("port = %d\n", port)
    fmt.Printf("other args: %+v\n", flag.Args())
}
```
运行./example5 -p=9000 foo=10 -bar 将会得到：
```sh
port = 9000
other args: [foo=10 -bar]
```
flag只要找到一个不包含的flag就会立即停止查询。

# 无效的flag参数
Go是一个强语言类型，所以如果我们传递一个string给一个int类型的flag，它将会提示我们：
```go
package main

import (
    "flag"
    "fmt"
)

func main() {
    var port int
    flag.IntVar(&port, "p", 8000, "specify port to use.  defaults to 8000")
    flag.Parse()

    fmt.Printf("port = %d", port)
}
```
运行程序./example6 -p=foo 得到的结果是：
```sh
invalid value “foo” for flag -p: strconv.ParseInt: parsing “foo”: invalid syntax
Usage of ./example6:
-p=8000: specify port to use.  defaults to 8000
```
flag不仅会提示我们输入错误，同时还会输出默认的使用方法。

# flag.Usage
flag包声明了一个Usage的方法。这样我们就可以输出我们想要输出的Usage了。
```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    flag.Usage = func() {
        fmt.Printf("Usage of %s:\n", os.Args[0])
        fmt.Printf("    example7 file1 file2 ...\n")
        flag.PrintDefaults()
    }
    flag.Parse()
}
```
运行./example7 –help 得到的结果是：
```sh
Usage of ./example7:
example7 file1 file2 …
```
# 获取输入
## fmt.Scanf
目前为止我们只是通过CLI输出了信息，但是不接受任何输入。我们可以基本的fmt.Scanf()来捕捉输入。
```go
package main

import "fmt"

func main() {
    var guessColor string
    const favColor = "blue"
    for {
        fmt.Println("Guess my favorite color:")
        if _, err := fmt.Scanf("%s", &guessColor); err != nil {
            fmt.Printf("%s\n", err)
            return
        }
        if favColor == guessColor {
            fmt.Printf("%q is my favorite color!", favColor)
            return
        }
        fmt.Printf("Sorry, %q is not my favorite color.  Guess again.\n", guessColor)
    }
}
```
## bufio.Scanner
fmt.Scanf 对于简单的输入很有效，但是有时候我们可能需要一整行的数据。
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    scanner := bufio.NewScanner(os.Stdin)
    for scanner.Scan() {
        line := scanner.Text()
        if line == "exit" {
            os.Exit(0)
        }
        fmt.Println(line) // Println will add back the final '\n'
    }
    if err := scanner.Err(); err != nil {
        fmt.Fprintln(os.Stderr, "reading standard input:", err)
    }
}
```
这是一个基本的echo程序，如果要退出直接输入exit即可。

# 一个基本的cat程序
你应该用过很多次cat程序了。我们将会把这篇博客学到的只是融合在一起构建一个基本的cat程序。
```go 
package main

import (
    "flag"
    "fmt"
    "io"
    "os"
)

func main() {
    flag.Usage = func() {
        fmt.Printf("Usage of %s:\n", os.Args[0])
        fmt.Printf("    cat file1 file2 ...\n")
        flag.PrintDefaults()
    }

    flag.Parse()
    if flag.NArg() == 0 {
        flag.Usage()
        os.Exit(1)
    }

    for _, fn := range flag.Args() {
        f, err := os.Open(fn); 
        if err != nil {
            panic(err)
        }
        _, err = io.Copy(os.Stdout, f)
        if err != nil {
            panic(err)
        }
    }
}
```
# 帮助
对于帮助我们在上面已经讲了，但是还没有明确的定义
```sh
-h
--h
–help
--help
```
上面这些都会触发help。

# 总结
本篇博客中只是讲了一些CLI的基本用法。如果想要学习更多，可以查看这些包的godoc
- flag
- bufio
- fmt

其他的命令行库
还有一些第三方库可以让写CLI程序更简单：
- comandante
- viper

---
[原文地址](https://thenewstack.io/cli-command-line-programming-with-go/)

[译文地址](http://www.cnblogs.com/hitandrew/p/5802521.html?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com)
