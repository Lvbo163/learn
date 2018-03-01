[原文地址]()

# go语言指针符号的*和&

<h2 name="目录">目录</h2>

[示例](#示例)

[解释](#解释)

[结果](#结果)

<h2 name="示例">示例</h2>
先看一段代码 先放一段代码，人工运行一下，看看自己能做对几题？

````go
package main

import "fmt"

func main() {
    var a int = 1 
    var b *int = &a
    var c **int = &b
    var x int = *b
    fmt.Println("a = ",a)
    fmt.Println("&a = ",&a)
    fmt.Println("*&a = ",*&a)
    fmt.Println("b = ",b)
    fmt.Println("&b = ",&b)
    fmt.Println("*&b = ",*&b)
    fmt.Println("*b = ",*b)
    fmt.Println("c = ",c)
    fmt.Println("*c = ",*c)
    fmt.Println("&c = ",&c)
    fmt.Println("*&c = ",*&c)
    fmt.Println("**c = ",**c)
    fmt.Println("***&*&*&*&c = ",***&*&*&*&*&c)
    fmt.Println("x = ",x)
}
````

<h2 name="解释">解释</h2>

<h3 name="理论">理论</h3>

&符号的意思是对变量取地址，如：变量a的地址是&a

\*符号的意思是对指针取值，如:\*&a，就是a变量所在地址的值，当然也就是a的值了

\*和 & 可以互相抵消,同时注意，\*&可以抵消掉，但&\*是不可以抵消的

a和\*&a是一样的，都是a的值，值为1 (因为\*&互相抵消掉了)

同理，a和\*&\*&\*&\*&a是一样的，都是1 (因为4个\*&互相抵消掉了)

<h3 name="展开">展开</h3>

因为有

````go
var b *int = &a
````

所以: a和\*&a和\*b是一样的，都是a的值，值为1 (把b当做&a看)

因为有

````go
var c **int = &b
````

所以, \*\*c和\*\*&b是一样的，把*&约去后

会发现\*\*c和\*b是一样的 (从这里也不难看出，*c和b也是一样的)

又因为上面得到的\*&a和\*b是一样的, 所以

\*\*c和\*&a是一样的，再次把*&约去后

\*\*c和a是一样的，都是1

不信你试试？

<h2 name="结果">结果</h2>

运行的结果内的地址值（0xc200开头的）可能会因不同机器运行而不同，你懂的

````go 
$ go run main.go 
a     =     1
&a     =     0xc200000018
*&a     =     1
b     =     0xc200000018
&b     =     0xc200000020
*&b     =     0xc200000018
*b     =     1
c     =     0xc200000020
*c     =     0xc200000018
&c     =     0xc200000028
*&c     =     0xc200000020
**c     =     1
***&*&*&*&c     =     1
x     =     1
````

两个符号抵消顺序 *&可以在任何时间抵消掉，但&*不可以被抵消的，因为顺序不对

fmt.Println("\*&a=",\*&a)  //成功抵消掉，打印出1，即a的值
fmt.Println("&\*a=",&\*a)  //无法抵消，会报错

[原文地址]: https://my.oschina.net/u/943306/blog/131269?hmsr=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com  "原文地址"
