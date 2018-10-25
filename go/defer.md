[原文地址1](https://www.cnblogs.com/vikings-blog/p/7099023.html)
[原文地址2](https://studygolang.com/articles/11522)

相关文档 [defer panic recover 异常处理](https://github.com/aimuke/learn/blob/master/go/defer%2C%20panic%2C%20recover.md)

# defer的使用

<!-- TOC -->
## 目录
- [规则](#规则)
    - [规则一 defer被声明时参数就会被解析](#规则一-defer被声明时参数就会被解析)
    - [规则二 多个defer执行顺序为先进后出](#规则二-多个defer执行顺序为先进后出)
    - [规则三 defer可以读取有名返回值](#规则三-defer可以读取有名返回值)
- [注意事项](#注意事项)
    - [对this参数的处理](#对this参数的处理)
    - [命名与匿名返回值处理](#命名与匿名返回值处理)
- [疑问?](#疑问)

<!-- /TOC -->

在golang当中，defer代码块会在函数调用链表中增加一个函数调用。这个函数调用不是普通的函数调用，而是会在函数正常返回，也就是return之后添加一个函数调用。因此，defer通常用来释放函数内部变量。

为了更好的学习defer的行为，我们首先来看下面一段代码:

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```
这段代码可以运行，但存在'安全隐患'。如果调用dst, err := os.Create(dstName)失败，则函数会执行return退出运行。但之前创建的src(文件句柄)没有被释放。 上面这段代码很简单，所以我们可以一眼看出存在文件未被释放的问题。 如果我们的逻辑复杂或者代码调用过多时，这样的错误未必会被及时发现。 而使用defer则可以避免这种情况的发生，下面是使用defer的代码:
```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```
通过defer，我们可以在代码中优雅的关闭/清理代码中所使用的变量。defer作为golang清理变量的特性，有其独有且明确的行为。以下是defer三条使用规则。

## 规则
### 规则一 defer被声明时参数就会被解析
> Each time a "defer" statement executes, the function value and parameters to the call are evaluated as usual and saved a new but the actual function is not invoked. 

每次一个defer申明执行的时候，函数和其参数都被解析，并且会被保存为一个新的值，但是函数并不会立马被执行. 这里需要注意的是，只有函数和参数是被作为新值修改了的。后面再对函数的变量进行修改都不会影响defer定义时函数的参数和值.

我们通过以下代码来解释这条规则:
```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
```
上面我们说过，defer函数会在return之后被调用。那么这段函数执行完之后，是不用应该输出1呢？结果输出的是0. why？

这是因为虽然我们在defer后面定义的是一个带变量的函数: fmt.Println(i). 在声明的时候 函数 fmt.Println 和变量 i 就已经确定了,并且被作为新值保存起来了，后面的修改不会对其产生影响。 换言之，上面的代码等同于下面的代码:
```go
func a() {
    i := 0
    defer fmt.Println(0) //因为i=0，所以此时就明确告诉golang在程序退出时，执行输出0的操作
    i++
    return
}
```
为了更为明确的说明这个问题，我们继续定义一个defer:
```go
func a() {
    i := 0
    defer fmt.Println(i) //输出0，因为i此时就是0
    i++
    defer fmt.Println(i) //输出1，因为i此时就是1
    return
}
```
通过运行结果，可以看到defer输出的值，就是定义时的值。而不是defer真正执行时的变量值

前面定义中说明了defer的函数和参数是在声明的时候就被解析了的，但是函数并没有真正执行。对于不是参数的变量是怎么样的呢？看下面的demo。
```go
func main() {
    for i :=0; i<5; i++ {
        p := i
        defer func(param int) {
            fmt.Println(param, i)
        }(p)
    }
}
```
输出为:
```ssh
4 5
3 5
2 5
1 5
0 5
```
可以看到 p 作为参数传入，是在定义的时候就确定了的, 输出的就是其定义时的值。但是 i 不是参数，只有在真正执行的时候才会获取，defer执行在函数return后执行，此时 i 已经变成了 5. 所以输出的结果里面所有的 i 输出的都是 5.

但为什么是先输出4，在输出0呢？ 看下面的规则二。

### 规则二 多个defer执行顺序为先进后出
当同时定义了多个defer代码块时，golang安装先定义后执行的顺序依次调用defer。这个很自然,后面的语句会依赖前面的资源,因此如果先前面的资源先释放了,后面的语句就没法玩了。我们用下面的代码加深记忆和理解:
```go
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)
    }
}
```
结合规则一，我们可以明确得知每个defer代码块应该输出什么值。按照先进后出的原则，我们可以看到依次输出了3210.

### 规则三 defer可以读取有名返回值
先看下面的代码:
```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```
输出结果是1还是2呢? 在开头的时候，我们说过defer是在return调用之后才执行的。 这里需要明确的是defer代码块的作用域仍然在函数之内，结合上面的函数也就是说，defer的作用域仍然在c函数之内。因此defer仍然可以读取c函数内的变量(如果无法读取函数内变量，那又如何进行变量清除呢....)。

当执行return 1 之后，i的值就是1. 此时此刻，defer代码块开始执行，对i进行自增操作。 因此输出2.

掌握了defer以上三条使用规则，那么当我们遇到defer代码块时，就可以明确得知defer的预期结果。


## 注意事项

### 对this参数的处理
这个大家用的都很频繁,但是go语言编程举了一个可能一不小心会犯错的例子.
```go
type Test struct {
    name string
}
func (t * Test) Close(){
    fmt.Println(t.name," closed");
}
func main(){
    ts:=[]Test{{"a"},{"b"},{"c"}}
    for _,t := range ts{
        defer  t.Close()
    }
}
```

```ssh
c  closed
c  closed
c  closed
```

这个输出并不会像我们预计的输出c b a,而是输出c c c

可是按照前面的go spec中的说明,应该输出c b a才对啊.

那我们换一种方式来调用一下.

```go
type Test struct {
    name string
}
func (t * Test) Close(){
    fmt.Println(t.name," closed");
}
func Close(t Test){
    t.Close()
}
func main(){
    ts:=[]Test{{"a"},{"b"},{"c"}}
    for _,t := range ts{
        defer Close(t)
    }
}
```
输出:
```ssh
c  closed
b  closed
a  closed
```

当然,如果你不想多写一个函数,也很简单,可以像下面这样,同样会输出c b a
```go
type Test struct {
    name string
}
func (t * Test) Close(){
    fmt.Println(t.name," closed");
}
func main(){
    ts:=[]Test{{"a"},{"b"},{"c"}}
    for _,t := range ts{
        t2:=t
        defer t2.Close()
    }
}
```
输出:
```ssh
c  closed
b  closed
a  closed
```

通过以上例子,我们在看一下对defer的说明
> Each time a "defer" statement executes, the function value and parameters to the call are evaluated as usualand saved anew but the actual function is not invoked. 

# 疑问?
这句话.可以得出下面的结论: 
> defer后面的语句在执行的时候,函数调用的参数会被保存起来,但是不执行.也就是复制了一份.但是并没有说struct这里的this指针如何处理,通过这个例子可以看出go语言并没有把这个明确写出来的this指针当作参数来看待.

### 命名与匿名返回值处理

先来运行下面两段代码：

- 匿名返回值的情况
```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("a return:", a()) // 打印结果为 a return: 0
}

func a() int {
    var i int
    defer func() {
        i++
        fmt.Println("a defer2:", i) // 打印结果为 a defer2: 2
    }()
    defer func() {
        i++
        fmt.Println("a defer1:", i) // 打印结果为 a defer1: 1
    }()
    return i
}
```
- 有名返回值的情况
```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("b return:", b()) // 打印结果为 b return: 2
}

func b() (i int) {
    defer func() {
        i++
        fmt.Println("b defer2:", i) // 打印结果为 b defer2: 2
    }()
    defer func() {
        i++
        fmt.Println("b defer1:", i) // 打印结果为 b defer1: 1
    }()
    return i // 或者直接 return 效果相同
}
```
先来假设出结论（这是正确结论），帮助大家理解原因：

多个 defer 的执行顺序为“后进先出/先进后出”；

所有函数在执行 RET 返回指令之前，都会先检查是否存在 defer 语句，若存在则先逆序调用 defer 语句进行收尾工作再退出返回；

匿名返回值是在 return 执行时被声明，有名返回值则是在函数声明的同时被声明，因此在 defer 语句中只能访问有名返回值，而不能直接访问匿名返回值；

return 其实应该包含前后两个步骤：第一步是给返回值赋值（若为有名返回值则直接赋值，若为匿名返回值则先声明再赋值）；第二步是调用 RET 返回指令并传入返回值，而 RET 则会检查 defer 是否存在，若存在就先逆序插播 defer 语句，最后 RET 携带返回值退出函数；

因此defer、return、返回值三者的执行顺序应该是：return最先给返回值赋值；接着 defer 开始执行一些收尾工作；最后 RET 指令携带返回值退出函数。

如何解释两种结果的不同：

上面两段代码的返回结果之所以不同，其实从上面的结论中已经很好理解了。

a()int 函数的返回值没有被提前声名，其值来自于其他变量的赋值，而 defer 中修改的也是其他变量（其实该 defer 根本无法直接访问到返回值），因此函数退出时返回值并没有被修改。

b()(i int) 函数的返回值被提前声名，这使得 defer 可以访问该返回值，因此在 return 赋值返回值 i 之后，defer 调用返回值 i 并进行了修改，最后致使 return 调用 RET 退出函数后的返回值才会是 defer 修改过的值。

下面我们再来看第三个例子，验证上面的结论：
```go
package main

import (
    "fmt"
)

func main() {
    c:=c()
    fmt.Println("c return:", *c, c) // 打印结果为 c return: 2 0xc082008340
}

func c() *int {
    var i int
    defer func() {
        i++
        fmt.Println("c defer2:", i, &i) // 打印结果为 c defer2: 2 0xc082008340
    }()
    defer func() {
        i++
        fmt.Println("c defer1:", i, &i) // 打印结果为 c defer1: 1 0xc082008340
    }()
    return &i
}
```
虽然 c()int 的返回值没有被提前声明，但是由于 c()int 的返回值是指针变量，那么在 return 将变量 i 的地址赋给返回值后，defer 再次修改了 i 在内存中的实际值，因此 return 调用 RET 退出函数时返回值虽然依旧是原来的指针地址，但是其指向的内存实际值已经被成功修改了。

即，我们假设的结论是正确的！

补充一条，defer声明时会先计算确定参数的值，defer推迟执行的仅是其函数体。
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    defer P(time.Now())
    time.Sleep(5e9)
    fmt.Println("main ", time.Now())
}

func P(t time.Time) {
    fmt.Println("defer", t)
    fmt.Println("P    ", time.Now())
}
```
输出结果：
```ssh
// main  2017-08-01 14:59:47.547597041 +0800 CST
// defer 2017-08-01 14:59:42.545136374 +0800 CST
// P     2017-08-01 14:59:47.548833586 +0800 CST
```
