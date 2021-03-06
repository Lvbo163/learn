[原文地址][]
# go语言中type的几种使用

2017-07-17 22:36

<h2 name="目录">目录</h2>

[1、结构体定义](#结构体定义)

[2、类型等价定义，相当于类型重命名](#类型等价定义)

[3、结构体内嵌匿名成员](#结构体内嵌匿名成员)

[4、定义接口类型](#定义接口类型)

[5、定义函数类型](#定义函数类型)

type是Go语法里的重要而且常用的关键字，type绝不只是对应于C/C++中的typedef。搞清楚type的使用，就容易理解go语言中的核心概念struct、interface、函数等的使用。以下我用例子代码总结描述，请特别留意代码中的注释。

<h2 name="结构体定义">1、定义结构体</h2>

````go

//结构体定义
type person struct {
	name string //注意后面不能有逗号
	age  int
}

func main() {

	//结构体初始化
	p := person{
		name: "taozs", //注意后面要加逗号
		age:  18,      //或者下面的}提到这儿来可以省略逗号
	}
	fmt.Println(p.name)

	//初始化字段不一定要全部指定，比如下面也是可以的，name默认取长度为0的空字符串
	p1 := person{
		age: 18,
	}
}
````

<h2 name="类型等价定义">2、类型等价定义，相当于类型重命名</h2>

````go
type name string
````
name类型与string等价

例子：

````go
type name string

func main() {
	var myname name = "taozs" //其实就是字符串类型
	l := []byte(myname)       //字符串转字节数组
	fmt.Println(len(l))       //字节长度
}

// 不过，要注意的是，type绝不只是用于定义一系列的别名。 还可以针对新类型定义方法。
// 上面的name类型可以像下面这样定义方法：
type name string

func (n name) len() int {
	return len(n)
}

func main() {
	var myname name = "taozs" //其实就是字符串类型
	l := []byte(myname)       //字符串转字节数组
	fmt.Println(len(l))       //字节长度
	fmt.Println(myname.len()) //调用对象的方法
}
````

<h2 name="结构体内嵌匿名成员">3、结构体内嵌匿名成员</h2>

````go
//结构体内嵌匿名成员定义
type person struct {
	string //直接写类型，匿名
	age    int
}

func main() {
	//结构体匿名成员初始化
	//可以省略部分字段，如：person{string: "taozs"}。也可以这样省略字段名：person{“taozs”, 18}，但必须写全了，不可以省略部分字段
	p := person{string: "taozs", age: 18} 
	
	//结构体匿名成员访问
	fmt.Println(p.string) //注意不能用强制类型转换（类型断言）：p.(string)
}

//以下是只有单个匿名成员的例子。
//结构体内嵌匿名成员定义
type person struct {
	string
}

func main() {
	//结构体匿名成员初始化
	p := person{string: "taozs"} //也可这样：person{"taozs"}
	//结构体匿名成员访问
	fmt.Println(p.string) //注意不能用强制类型转换（类型断言）：p.(string)
}

````

<h2 name="定义接口类型">4、定义接口类型</h2>

````go
package main

import (
	"fmt"
)

//接口定义
type Personer interface {
	Run()
	Name() string
}

//实现接口，注意实现接口的不一定只是结构体，也可以是函数对象，参见下面第5条
type person struct {
	name string
	age  int
}

func (person) Run() {
	fmt.Println("running...")
}

//接收参数person不可以是指针类型，否则不认为是实现了接口
func (p person) Name() string {
	return p.name
}

func main() {
	//接口类型的变量定义
	var p Personer
	fmt.Println(p) //值<nil>
	//实例化结构体，并赋值给interface
	p = person{"taozs", 18} //或者：&person{"taozs", 18}
	p.Run()
	fmt.Println(p.Name())
	var p2 person = p.(person) //类型断言，接口类型断言到具体类型
	fmt.Println(p2.age)
}

````

另外，类型断言返回值也可以有第二个bool值，表示断言是否成功，如下：

`````go
if p2, ok := p.(person); ok { //断言成功ok值为true
	fmt.Println(ok)
	fmt.Println(p2.age)
}
`````
<h2 name="定义函数类型">5、定义函数类型</h2>

以下是定义一个函数类型handler

````go
type handler func(name string) int
````

针对这个函数类型可以再定义方法，如：

````go
func (h handler) add(name string) int {
    return h(name) + 10
}
````

下面让我们详细看一下例子，其中涉及了函数、函数的方法、结构体方法、接口的使用。

````go
package main

import (
	"fmt"
)

//定义接口
type adder interface {
	add(string) int
}

//定义函数类型
type handler func(name string) int

//实现函数类型方法
func (h handler) add(name string) int {
	return h(name) + 10
}

//函数参数类型接受实现了adder接口的对象（函数或结构体）
func process(a adder) {
	fmt.Println("process:", a.add("taozs"))
}

//另一个函数定义
func doubler(name string) int {
	return len(name) * 2
}

//非函数类型
type myint int

//实现了adder接口
func (i myint) add(name string) int {
	return len(name) + int(i)
}

func main() {
	//注意要成为函数对象必须显式定义handler类型
	var my handler = func(name string) int {
		return len(name)
	}
	//以下是函数或函数方法的调用
	fmt.Println(my("taozs")) //调用函数
	fmt.Println(my.add("taozs")) //调用函数对象的方法
	fmt.Println(handler(doubler).add("taozs")) //doubler函数显式转换成handler函数对象然后调用对象的add方法

	//以下是针对接口adder的调用
	process(my) //process函数需要adder接口类型参数
	process(handler(doubler)) //因为process接受的参数类型是handler，所以这儿要强制转换
	process(myint(8)) //实现adder接口不仅可以是函数也可以是结构体
}

````

[原文地址]: http://www.sohu.com/a/157951162_99930294 "原文地址"
