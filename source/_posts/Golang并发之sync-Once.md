---
title: Golang并发之sync.Once
toc_number: true
copyright: true
date: 2019-12-22 22:39:57
categories:
- Golang学习笔记
tags:
- golang
---

# 使用

sync.Once 可以控制函数只能被调用一次，不能多次重复调用。

例如如下代码：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	once := sync.Once{}
	n := 10
	for i := 0; i < n; i++ {
		once.Do(f)
	}
}

func f() {
	fmt.Println("Hello World")
}
```

虽然在 for 循环中多次执行了 once.Do(f) ，但是最终却只执行了一次f 函数而已。

<!--more-->

这是什么原因呢？查看一下 sync.Once 的源码如下：

```go
type Once struct {
    // 执行标记
	done uint32
    // 互斥锁
	m    Mutex
}

func (o *Once) Do(f func()) {
    // 原子操作判断o.done是否为1，若为1，表示f已经执行过，直接返回
	if atomic.LoadUint32(&o.done) == 0 {
		// Outlined slow-path to allow inlining of the fast-path.
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
     //加锁，保证互斥访问
	o.m.Lock()
	defer o.m.Unlock()
    //这里需要再次重新判断下，因为 atomic.LoadUint32取出状态值到  o.m.Lock() 之间是有可能存在其它gotoutine改变status的状态值的
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

可以看到 sync.Once 只执行函数一次的原理类似于单例模式中的双重检查锁，因此 sync.Once 最好的应用场景就是在单例模式中使用它。

如下就可以每次调用 singleton.Instance() 都是返回同一个单例

```go
package singleton

import (
    "fmt"
    "sync"
)

// 单例结构体
type object struct {
}

var once sync.Once
var obj *object //单例指针

//公开方法 外包调用
func Instance() *object {
    once.Do(getObj)
    return obj
}

func getObj() {
    if obj == nil {
        obj = new(object)
        //可以做其他初始化事件
    }
}
```

