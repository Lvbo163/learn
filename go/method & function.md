[原文地址](https://blog.csdn.net/D_Guco/article/details/53575067)

## golang 函数以及函数和方法的区别

在接触到go之前，我认为函数和方法只是同一个东西的两个名字而已（在我熟悉的c/c++，python，java中没有明显的区别），但是在golang中者完全是两个不同的东西。官方的解释是，方法是包含了接收者的函数。到底什么意思呢。

## func

首先函数的格式是固定的，**func＋函数名＋ 参数 ＋ 返回值（可选） ＋ 函数体**。 

```go
func main() {
  fmt.Println("Hello go")
}
```

### main & init

在golang中有两个特殊的函数，main函数和init函数。

- main函数不用介绍在所有语言中都一样，它作为一个程序的入口，只能有一个。

- init函数在每个package是可选的，可有可无，甚至可以有多个(但是强烈建议一个package中一个init函数)，init函数在你导入该package时程序会自动调用init函数，所以init函数不用我们手动调用,l另外它只会被调用一次，因为当一个package被多次引用时，它只会被导入一次。

```go
package mypackage
 
import (
	"fmt"
)
 
var I int
 
func init() {
	I = 0
	fmt.Println("Call mypackage init1")
}
 
func init() {
	I = 1
	fmt.Println("Call mypackage init2")
}
package main
 
import (
	"demo/mypackage"
	"fmt"
)
 
func main() {
	fmt.Println("Hello go.... I = ", mypackage.I)
}
```

运行结果:

```ssh
go run main.go
Call mypackage init1
Call mypackage init2
Hello go.... I = 1
```

我们可以看到，程序为我们自动调用了两个init函数，并且是按照顺序调用的。

### method

下面来看方法。

```go
package main
 
import "fmt"
 
type myint int
 
//乘2 指针 - 类似于类的非static方法，修改类的成员值
func (p *myint) mydouble() int {
	*p = *p * 2
	return 0
}
 
//平方 值 - 类似于类的static 方法，不改变实例的成员
func (p myint) mysquare() int {
	p = p * p
	fmt.Println("mysquare p = ", p)
	return 0
}
 
func main() {
	var i myint = 2
	i.mydouble()
	fmt.Println("i = ", i)
	i.mysquare()
	fmt.Println("i = ", i)
}
```

运行结果：

```ssh
go run main.go
i = 4
mysquare p = 16
i = 4
```

### 总结

我们可以看到方法和函数的区别，方法在func关键字后是接收者而不是函数名，接收者可以是自己定义的一个类型，这个类型可以是struct，interface，甚至我们可以重定义基本数据类型。我们可以给他一些我们想要的方法来满足我们的实际工程中的需求，就像上面一样我重定义了int并给了它一个乘2和平法的方法。

**这里要注意一个细节，接收者是指针或非指针。当接收者为指针时，通过方法可以改变该接收者的属性，但是非指针类型不行。
这里的接收者和c++中的this指针相似，可以把接受者当作一个class，方法就是类的成员函数。当接收者为指针类型时就是c++中的非const成员函数；为非指针时就是const成员函数，不能通过此方法改变累的成员变量。**
