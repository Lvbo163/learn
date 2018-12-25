# [Golang中WaitGroup使用的一点坑](https://liudanking.com/golang/golang-waitgroup-usage/)

Golang 中的 WaitGroup 一直是同步 goroutine 的推荐实践。自己用了两年多也没遇到过什么问题。直到一天午睡后，同事扔过来一段奇怪的代码：

# 坑1
```go
package main
 
import (
    "log"
 
    "sync"
)
 
func main() {
    wg := sync.WaitGroup{}
 
    for i := 0; i &lt; 5; i++ {
        go func(wg sync.WaitGroup, i int) {
            wg.Add(1)
            log.Printf("i:%d", i)
            wg.Done()
        }(wg, i)
    }
 
    wg.Wait()
 
    log.Println("exit")
}
```
 
撇了一眼，觉得没什么问题。然而，它的运行结果是这样：

```ssh
2016/11/27 15:12:36 exit
[Finished in 0.7s]
```
或这样：
```ssh
2016/11/27 15:21:51 i:2
2016/11/27 15:21:51 exit
[Finished in 0.8s]
```
或这样：
```go
2016/11/27 15:22:51 i:3
2016/11/27 15:22:51 i:2
2016/11/27 15:22:51 exit
[Finished in 0.8s]
```
 
一度让我以为手上的 mac 也没睡醒……
这个问题如果理解了 WaitGroup 的设计目的就非常容易 fix 啦。因为 WaitGroup 同步的是 goroutine, 而上面的代码却在 goroutine 中进行 Add(1) 操作。因此，可能在这些 goroutine 还没来得及 Add(1) 已经执行 Wait 操作了。

于是代码改成了这样：

# 坑2
```go
package main

import (
    "log"

    "sync"
)

func main() {
    wg := sync.WaitGroup{}

    for i := 0; i &lt; 5; i++ {
        wg.Add(1)
        go func(wg sync.WaitGroup, i int) {
            log.Printf("i:%d", i)
            wg.Done()
        }(wg, i)
    }

    wg.Wait()

    log.Println("exit")
}
```
 
然而，mac 又睡了过去，而且是睡死了过去：
```go
2016/11/27 15:25:16 i:1
2016/11/27 15:25:16 i:2
2016/11/27 15:25:16 i:4
2016/11/27 15:25:16 i:0
2016/11/27 15:25:16 i:3
fatal error: all goroutines are asleep - deadlock!
```
wg 给拷贝传递到了 goroutine 中，导致只有 Add 操作，其实 Done操作是在 wg 的副本执行的。因此 Wait 就死锁了。于是代码改成了这样：

# 填坑
```go
package main

import (
    "log"

    "sync"
)

func main() {
    wg := &amp;sync.WaitGroup{}

    for i := 0; i &lt; 5; i++ {
        wg.Add(1)
        go func(wg *sync.WaitGroup, i int) {
            log.Printf("i:%d", i)
            wg.Done()
        }(wg, i)
    }

    wg.Wait()

    log.Println("exit")
}
```
至此，午睡终于睡醒了。Sigh…

(其实还没有结束，看下面的补充)

---
# 补充：
golang 给出的 demo 中并没有将 waitgroup 作为参数掺入，而是直接在main routine中使用。 修改后的代码应该是这样的：
```go
package main

import (
    "log"

    "sync"
)

func main() {
    wg := sync.WaitGroup{}

    for i := 0; i &lt; 5; i++ {
        wg.Add(1)
        go func(i int) {    // 不需要将wg作为参数传入，直接使用main作用域的变量
            defer wg.Done()     // 应该将wg.Done() 放到defer中，而不是放到代码最后，避免中间代码出现错误退出，而routine没有结束
            log.Printf("i:%d", i)
        }(wg, i)
    }

    wg.Wait()

    log.Println("exit")
}
```
修改点如上见注释:
- 不需要将wg作为参数传入，直接使用main作用域的变量
- 应该将wg.Done() 放到defer中，而不是放到代码最后，避免中间代码出现错误退出，而routine没有结束
