[原文地址](http://www.jb51.net/article/74001.htm)

# Go语言的os包中常用函数初步归纳

<h2 name="目录">目录</h2>

[(1)os.Getwd](#Getwd)

[(2)os.Getenv](#Getenv)

[(3)os.get](#get)

[(4)os.Chdir()](#Chdir)

[(5)os.Stat()](#Stat)

[(6)os.Chmod()](#Chmod)

[(7)os.Chtime()](#Chtime)

[(8)os.Environ()](#Environ)

[(9)os.Exit()](#Exit)

[(10)os.Expand()](#Expand)

[(11)os.ExpandEnv()](#ExpandEnv)

[(12)os.Hostname()](#Hostname)

这篇文章主要介绍了Go语言的os包中常用函数初步归纳,用于一些和系统交互功能的实现,需要的朋友可以参考下

<h2 name="Getwd">(1)os.Getwd</h2>

函数原型是func Getwd() (pwd string, err error) 返回的是路径的字符串和一个err信息，为什么先开这个呢？因为我看os的包的时候第一个是Chkdir这个包，但是你不知道当前目录怎么知道改变目录了呢？所以先说Getwd() 函数demo

````
import (
 "fmt"
 "os"
)
func main() {
 dir, _ := os.Getwd()
 fmt.Println("当前的目录是:", dir)  //当前的目录是: D:\test 我的环境是windows 如果linix 就是/xxx/xxx
}
````

<h2 name="Getenv">(2)os.Getenv</h2>

os.Getenv()获取系统的环境变量，函数原型是func Getenv(key string) string输入的是一个string的环境变量名称，返回的是值

````
import (
 "fmt"
 "os"
)
func main() {
 path := os.Getenv("GOPATH")
 fmt.Println("环境变量GOPATH的值是:", path) //windows下 环境变量PATH的值是: D:\test;C:\Go\bin; linux 环境变量GOPATH的值是: /data/goweb
}
````

<h2 name="get">(3)os.get</h2>

下边的get信息 如果没有：=的就是返回的都是int一般很少用到的 我就给注释了做什么的？然后windows和linux结果是什么？

````
 fmt.Println(os.Getegid())      windows -1  linux  0     //调用者的group的id
 fmt.Println(os.Geteuid())     windows -1  linux  0     //用户的uid 
 fmt.Println(os.Getgid())      windows -1  linux  0     //调用者的gid的id
 g, _ := os.Getgroups()        
 fmt.Println(g)                windows []  linux  []    //返回的是一个[]int的切片 显示调用者属于组的一系列id
 fmt.Println(os.Getpagesize())  windows 4096linux  4096  //windows里边叫做虚拟内存 linux里边叫做swap
 fmt.Println(os.Getppid())      windows -1  linux  8621  //调用者的组的进程id
 fmt.Println(os.Getuid())    windows -1  linux  0  //调用者的数字用户id
````

<h2 name="Chdir">(4)os.Chdir()</h2>

这个函数的原型是func Chdir(dir string) error 输入字符类型，返回的是错误结果，如果改变成功了error=nil

````
import (
 "fmt"
 "os"
)
func main() {
 fmt.Println(os.Getwd())              //显示当前的目录 D:\test <nil>
 fmt.Println(os.Chdir("D:/test/src")) //返回<nil>正确切换目录了
 fmt.Println(os.Getwd())              //切换后的目录D:\test\src <nil>
}
````

<h2 name="Stat">(5)os.Stat()</h2>

获取文件的信息，函数函数的原型func Stat(name string) (fi FileInfo, err error)输出是文件的名称返回一个FileInfo的接口和err信息，上一个分析ioutil的时候我们就介绍FileInfo的接口类型了

````
type FileInfo interface {
    Name() string       // 文件的名称
    Size() int64        // 唱过文件的大小
    Mode() FileMode     // 文件的权限
    ModTime() time.Time // 时间
    IsDir() bool        // 是否是目录
    Sys() interface{}   // 基础数据源接口(can return nil)
}
import (
 "fmt"
 "os"
)
func main() {
 filemode, _ := os.Stat("widuu.go")  
 fmt.Println(filemode.Mode())        //获取权限 linux 0600
}
````

<h2 name="Chmod">(6)os.Chmod()</h2>

原型是func Chmod(name string, mode FileMode) error改变文件的属性 譬如读写，linux上的0755这样大家可以理解了吧

````
import (
 "fmt"
 "os"
)
func main() {
 filemode, _ := os.Stat("widuu.go")  
 fmt.Println(filemode.Mode())        //获取权限 linux 0600
 err := os.Chmod("widuu.go", 0777)   //改变的是文件的权限
 if err!=nil{
  fmt.Println("修改文件权限失败")
 }
 filemode, _ = os.Stat("widuu.go")  
 fmt.Println(filemode.Mode())        //获取权限是0777

}
````

<h2 name="Chtime">(7)os.Chtime()</h2>

函数的原形是func Chtimes(name string, atime time.Time, mtime time.Time) error 输入string的文件的名称 访问时间 创建时间 返回的是error接口信息

````
import (
 "fmt"
 "os"
 "time"
)
func main() {
 err := os.Chtimes("2.go", time.Now(), time.Now())  //改变时间
 if err != nil {
  fmt.Println(err)
 }
 fi, _ := os.Stat("2.go")
 fmt.Println(fi.ModTime())   //输出时间 2013-12-29 20:46:23.0005257 +0800 +0800
}
````

<h2 name="Environ">(8)os.Environ()</h2>

作用是获取系统的环境变量，函数原形是func Environ() []string返回是环境变量的[]string切片，说道这个就要和其他的一起说明了，那就是os.ClearEnv()清空环境变量

````go
func main() {
 data := os.Environ() //输出之前的环境变量 APPDATA=C:\Users\xiaolvge\AppData\Roaming CLASSPATH=.;D:\java\jdk1.6.0_38…………
 fmt.Println(data)
 os.Clearenv() //清空环境变量
 data = os.Environ()
 fmt.Println(data) //输出[]string类型的切片[]
}
````

<h2 name="Exit">(9)os.Exit()</h2>

就是中断程序返回自定义的int类型的代码，函数运行是func Exit(code int)输入一个int的值就可以了

````
import (
 "fmt"
 "os"
)
func main() {
 func() {
  for {
   fmt.Println("这个是匿名函数")
   os.Exit(1)    //输出exit status 1中断操作
  }
 }()
}
````

<h2 name="Expand">(10)os.Expand()</h2>

是一个回调函数替换的方法，函数的原形是func Expand(s string, mapping func(string) string) string 输入的是一个string。对应的是func(string)string的替换字符串的方法，如果没有字符就替换为空

````
import (
 "fmt"
 "os"
)
func main() {
 mapping := func(s string) string {
  m := map[string]string{"widuu": "www.jb51.net", "xiaowei": "widuu"}
  return m[s]
 }
 data := "hello $xiaowei blog address $widuu"
 fmt.Printf("%s", os.Expand(data, mapping)) //输出hello widuu blog address www.jb51.net}
````

<h2 name="ExpandEnv">(11)os.ExpandEnv()</h2>

把字符串的s替换成环境变量的内容，函数的原形是func ExpandEnv(s string) string，输入的当然是要替换的字符，输出的当然还是字符串了

````go
import (
 "fmt"
 "os"
)
func main() {
 data := "GOBIN PATH $GOBIN"
 fmt.Println(os.ExpandEnv(data)) //输出我本地的环境变量的GOBIN的地址GOBIN PATH C:\Go\bin
}
````

<h2 name="Hostname">(12)os.Hostname()</h2>

返回主机的HostName(),函数的原形是func Hostname() (name string, err error)返回主机名字和一个error的接口信息

````go
import (
 "fmt"
 "os"
)
func main() {
 data, _ := os.Hostname()
 fmt.Println(data) //我是windows环境下返回我的win的主机名 WIDUU
}
````
