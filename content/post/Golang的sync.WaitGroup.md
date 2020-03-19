---
title: Golang的sync.WaitGroup
date: 2019-12-22 10:37:28
categories: [Go学习笔记]
tags: [golang]
---

# 使用

假设我们需要在主函数中调用协程打印数据，

很容易我们能想写出下面的代码

```go
package main

import "fmt"

func main() {
	n := 100
	for i := 0; i < n; i++ {
		go fmt.Println("Hello World!")
	}
}
```

本来我们的要求是需要打印 100 个 "Hello World" 在控制台上，但是实际运行起来会发现每次都是少那么几个，这是什么原因呢？因为在 main 函数执行完毕之后由它所派生出的协程也将会全部给操作系统给杀死，而不会再继续执行其相应的操作了。

由于以上原因，我们很快就能想到解决方案。既然是由于 main 函数的结束所产生的问题，那么我们在最后让 main 函数不直接结束不就好了？修改代码如下



```go
package main

import (
	"fmt"
	"time"
)

func main() {
	n := 100
	for i := 0; i < n; i++ {
		go fmt.Println("Hello World")
	}
	time.Sleep(1 * time.Second)
}
```

尝试运行发现，emmm，完美，成功输出100个 "Hello World" 了。但是这段代码真的就完美了吗？这里仅仅是打印100个 "Hello World" ，如果我想打印多一点的 "Hello World"可以吗？假设是为 10000甚至是100000000个，运行程序后发现，对不起，并不能如我所愿。

这是什么原因呢？这段代码最大的问题就是你无法控制程序运行的时间，如果打印所有的 "Hello World" 在1秒内运行结束了，那么代码执行结果正确，如果中间出现了一点啥问题，导致一秒之内程序未能打印全部的 "Hello World" ，那么就会出现执行结果错误。作为写代码的，当然不能将程序的执行成败当做一个随机事件来产生，那么有没有什么更好的方法呢？

可以考虑使用管道来完成上述操作

```go
package main

import (
	"fmt"
)

func main() {
	n := 100
	c := make(chan bool, n)
	for i := 0; i < n; i++ {
		go func() {
			fmt.Println("Hello World")
			c <- true
		}()
	}
	for i := 0; i < n; i++ {
		<-c
	}
}
```

上面的代码使用了管道来进行协程间的通信，首先可以肯定的是使用管道是能达到我们的目的的，而且不但能达到目的，还能十分完美的达到目的。

但是这种方法有什么坏处呢？

首先，管道在这里显得有些大材小用，因为它被设计出来不仅仅只是在这里用作简单的同步处理。

在这里使用管道实际上是不合适的。而且假设我们有一万、十万甚至更多的for循环，也要申请同样数量大小的管道出来，对内存也是不小的开销。

所以，还有没有什么更好的方法呢？

最后，让我们有请本次的主角 `sync.WaitGroup` 闪亮登场！

 Golang 中的 `sync.WaitGruop` 是属于 sync 包下的一个同步处理的结构体，它用来做 go routine 的等待，当启动多个 go routime的时候，通过  `WaitGruop` 可以等待所有 go routime 程序结束后再执行后面的代码逻辑。

`WaitGroup` 对象内部有一个计数器，最初从0开始，它有三个方法：`Add(delta int), Done(), Wait()` 用来控制计数器的数量。`Add(n)` 把计数器加n (n可以为负数)，`Done()` 调用 `Add(-1)` ，`wait()` 会阻塞代码的运行，直到计数器地值减为0。

修改代码如下：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	n := 100
	wg := sync.WaitGroup{}
	wg.Add(n)
	for i := 0; i < n; i++ {
		go func() {
			fmt.Println("Hello World")
			wg.Done()
		}()
	}
	wg.Wait()
}
```

这里首先把`wg` 计数设置为100， 每个for循环运行完毕都把计数器减一，主函数中使用`Wait()` 一直阻塞，直到wg为零——也就是所有的100个for循环都运行完毕。相对于使用管道来说，`WaitGroup` 轻巧了许多。

# 注意事项

1. 计数器不能为负值

`WaitGroup` 内的计数器不能变成负值，否则会报 panic 错误

2. WaitGroup对象不是一个引用类型

`WaitGroup` 对象不是一个引用类型，在通过函数传值的时候需要使用地址：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	n := 100
    wg := sync.WaitGroup{}
    wg.Add(n)
    for i := 0; i < n; i++ {
        go f(&wg)
    }
    wg.Wait()
}

// 一定要通过指针传值，不然进程会进入死锁状态
func f(wg *sync.WaitGroup) { 
    fmt.Println("Hello World")
    wg.Done()
}
```