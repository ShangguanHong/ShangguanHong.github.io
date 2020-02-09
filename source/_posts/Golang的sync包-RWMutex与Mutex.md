---
title: Golang的sync包-RWMutex与Mutex
toc_number: true
copyright: true
date: 2019-12-26 20:46:29
categories:
- Golang学习笔记
tags:
- golang
---

# 介绍

Golang 中的 sync 包中实现了两种锁，一种是 Mutex(互斥锁)，另一种是RWMutex(读写锁)，其中 RWMutex 是基于 Mutex 实现的

# Mutex

Mutex 是互斥锁，Lock() 是加锁，Unlock() 是解锁，在一个 goruntine 对 Mutex 进行上锁之后，其他的 goruntine 只能等到该 goruntine 对该 Mutex 解锁之后才能在对其进行加锁操作。  

示例如下

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
    // 保证四个goroutine执行完之前主函数不会退出
	wg := sync.WaitGroup{}
	wg.Add(4)
	lock := sync.Mutex{}
	fmt.Println("This lock is locked")
	lock.Lock()
	for i := 0; i < 4; i++ {
		go func(i int, wg *sync.WaitGroup) {
			fmt.Println(i, "Not lock")
			lock.Lock()
			fmt.Println(i, "get the lock")
			time.Sleep(time.Second)
			fmt.Println(i, "unlock the lock")
			lock.Unlock()
			wg.Done()
		}(i, &wg)
	}
	time.Sleep(time.Second)
	fmt.Println("This lock is unlock")
	lock.Unlock()
	wg.Wait()
}
```

<!--more-->

执行结果

![image-20191226210905112](Golang%E5%B9%B6%E5%8F%91%E4%B9%8Bsync-RWMutex%E4%B8%8Esync-Mutex/image-20191226210905112.png)

使用 mutex 注意事项如下

1. 在一个 goruntine 对 Mutex 进行上锁之后，其他的 goruntine 只能等到该 goruntine 对该 Mutex 解锁之后才能在对其进行加锁操作

2. 一个 goruntine 对 Mutex 加锁之后，不能再继续对其进行加锁，需要解锁之后才能在进行加锁，否则会形成死锁，报错如下

   fatal error: all goroutines are asleep - deadlock!

3. 在 Lock() 之前使用 Unlock() 会导致 panic 异常，信息如下

   fatal error: sync: unlock of unlocked mutex

4. 适用于读写不确定，并且只有一个读或者写的场景

# RWMutex

RWMutex 是读写锁， 可以加多个读写和一个写锁。在读锁被占用的时候会阻止写锁，但是不会阻止读锁，多个 goruntine 可以共享读锁， 写锁会阻止写锁和读写，写锁一次只能被一个 goruntine 所占用。

RLock() 是加读锁， RUnlock() 是解读锁。

示例如下

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	wg := sync.WaitGroup{}
	wg.Add(4)
	lock := sync.RWMutex{}
	fmt.Println("This lock is locked")
	lock.RLock()
	for i := 0; i < 4; i++ {
		go func(i int, wg *sync.WaitGroup) {
			lock.RLock()
			fmt.Println(i, "get the lock")
			time.Sleep(time.Second)
			fmt.Println(i, "unlock the lock")
			lock.RUnlock()
			wg.Done()
		}(i, &wg)
	}
	time.Sleep(time.Second)
	fmt.Println("This lock is unlock")
	lock.RUnlock()
	wg.Wait()
}
```

结果

![image-20191226213240734](Golang%E5%B9%B6%E5%8F%91%E4%B9%8Bsync-RWMutex%E4%B8%8Esync-Mutex/image-20191226213240734.png)

可以看到在主函数释放读锁之前，其他的 goruntine 也一样可以获取到读锁

注意事项：

1. RLock() 加读锁时，如果存在写锁，则无法加读锁；当只有读锁或者没有锁时，可以加读锁，读锁可以加载多个
2. RUnlock() 解读锁，RUnlock() 撤销当前 RLock() 调用，对于其他同时存在的读锁没有效果
3. 在没有读锁的情况下调用 RUnlock() 会导致 panic 错误
4. RUnlock() 的个数不得多余 RLock()，否则会导致 panic 错误

至于写锁其实就和 Mutex 的用法一样，唯一的不同就是如果在加写锁之前已经有其他的读锁和写锁，则 Lock() 会阻塞直到该锁可用，为确保该锁可用，已经阻塞的 Lock() 调用会从获得的锁中排除新的读取器，即写锁权限高于读锁，有写锁时优先进行写锁定。

RWMutex使用错误示范如下

```go
package main

import "sync"

func main() {
	rwMutex := sync.RWMutex{}
    // 1. 未解写锁之前加写锁，发生死锁
	rwMutex.Lock()
	rwMutex.Lock()
    
    // 2. 未解写锁之前加读锁，发生死锁
    rwMutex.Lock()
    rwMutex.RLock()
    
    // 3. 在加读锁之前解读锁，抛出panic
    rwMutex.RUnlock()
    
    //4. 解读锁个数大于加读锁个数，抛出panic
    rwmutex.RLock()
    rwmutex.RLock()
    rwmutex.RUnlock()
    rwmutex.RUnlock()
    rwmutex.RUnlock()
}
```

