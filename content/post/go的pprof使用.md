---
title: go的pprof使用
date: 2020-02-26 22:01:46
categories: [Go学习笔记]
tags: [golang]
---

原文地址：[Golang 大杀器之性能剖析 PProf](https://book.eddycjy.com/golang/tools/go-tool-pprof.html)

# 前言

写了几吨代码，实现了几百个接口。功能测试也通过了，终于成功的部署上线了

结果，性能不佳，什么鬼？😭

# 想做性能分析

## PProf

想要进行性能优化，首先瞩目在 Go 自身提供的工具链来作为分析依据，本文将带你学习、使用 Go 后花园，涉及如下：

- runtime/pprof：采集程序（非 Server）的运行数据进行分析
- net/http/pprof：采集 HTTP Server 的运行时数据进行分析

## 是什么

pprof 是用于可视化和分析性能分析数据的工具

pprof 以 [profile.proto](https://github.com/google/pprof/blob/master/proto/profile.proto) 读取分析样本的集合，并生成报告以可视化并帮助分析数据（支持文本和图形报告）

profile.proto 是一个 Protocol Buffer v3 的描述文件，它描述了一组 callstack 和 symbolization 信息， 作用是表示统计分析的一组采样的调用栈，是很常见的 stacktrace 配置文件格式

## 支持什么使用模式

- Report generation：报告生成
- Interactive terminal use：交互式终端使用
- Web interface：Web 界面

## 可以做什么

- CPU Profiling：CPU 分析，按照一定的频率采集所监听的应用程序 CPU（含寄存器）的使用情况，可确定应用程序在主动消耗 CPU 周期时花费时间的位置
- Memory Profiling：内存分析，在应用程序进行堆分配时记录堆栈跟踪，用于监视当前和历史内存使用情况，以及检查内存泄漏
- Block Profiling：阻塞分析，记录 goroutine 阻塞等待同步（包括定时器通道）的位置
- Mutex Profiling：互斥锁分析，报告互斥锁的竞争情况

## 一个简单的例子

我们将编写一个简单且有点问题的例子，用于基本的程序初步分析

demo.go文件内容：

```go
package main

import (
    "log"
    "net/http"
    _ "net/http/pprof"
)

var datas []string

func main() {
    go func() {
        for {
            log.Println(Add("Hello World!"))
        }
    }()
    http.ListenAndServe("0.0.0.0:6060", nil)
}

func Add(str string) string {
    data := []byte(str)
    sData := string(data)
    datas = append(datas, sData)
    return sData
}
```

运行这个文件，你的 HTTP 服务会多出 /debug/pprof 的 endpoint 可用于观察应用程序的情况

## 分析

### 一、通过 Web 界面

查看当前总览：访问 `http://127.0.0.1:6060/debug/pprof/`

```
/debug/pprof/

Types of profiles available:
Count	Profile
26	allocs
0	block
0	cmdline
4	goroutine
26	heap
0	mutex
0	profile
8	threadcreate
0	trace
full goroutine stack dump
Profile Descriptions:

allocs: A sampling of all past memory allocations
block: Stack traces that led to blocking on synchronization primitives
cmdline: The command line invocation of the current program
goroutine: Stack traces of all current goroutines
heap: A sampling of memory allocations of live objects. You can specify the gc GET parameter to run GC before taking the heap sample.
mutex: Stack traces of holders of contended mutexes
profile: CPU profile. You can specify the duration in the seconds GET parameter. After you get the profile file, use the go tool pprof command to investigate the profile.
threadcreate: Stack traces that led to the creation of new OS threads
trace: A trace of execution of the current program. You can specify the duration in the seconds GET parameter. After you get the trace file, use the go tool trace command to investigate the trace.
```

这个页面有许多子页面，咱们继续深究下去，看看可以得到什么？

- cpu（CPU Profiling）: `$HOST/debug/pprof/profile`，默认进行 30s 的 CPU Profiling，得到一个分析用的 profile 文件
- block（Block Profiling）：`$HOST/debug/pprof/block`，查看导致阻塞同步的堆栈跟踪
- goroutine：`$HOST/debug/pprof/goroutine`，查看当前所有运行的 goroutines 堆栈跟踪
- heap（Memory Profiling）: `$HOST/debug/pprof/heap`，查看活动对象的内存分配情况
- mutex（Mutex Profiling）：`$HOST/debug/pprof/mutex`，查看导致互斥锁的竞争持有者的堆栈跟踪
- threadcreate：`$HOST/debug/pprof/threadcreate`，查看创建新OS线程的堆栈跟踪

### 二、通过交互式终端使用

（1）go tool pprof http://localhost:6060/debug/pprof/profile?seconds=60

```sh
$ go tool pprof http://localhost:6060/debug/pprof/profile?seconds=60
Fetching profile over HTTP from http://localhost:6060/debug/pprof/profile?seconds=60
Saved profile in C:\Users\fate\pprof\pprof.samples.cpu.001.pb.gz
Type: cpu
Time: Feb 26, 2020 at 10:15pm (CST)
Duration: 1mins, Total samples = 1.06mins (106.05%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```

执行该命令后，需等待 60 秒（可调整 seconds 的值），pprof 会进行 CPU Profiling。结束后将默认进入 pprof 的交互式命令模式，可以对分析的结果进行查看或导出。具体可执行 `pprof help` 查看命令说明

```sh
(pprof) top
Showing nodes accounting for 50.42s, 79.02% of 63.81s total
Dropped 154 nodes (cum <= 0.32s)
Showing top 10 nodes out of 52
      flat  flat%   sum%        cum   cum%
    41.54s 65.10% 65.10%     42.19s 66.12%  runtime.cgocall
     1.42s  2.23% 67.32%      1.78s  2.79%  log.itoa
     1.39s  2.18% 69.50%      3.26s  5.11%  runtime.mallocgc
     1.21s  1.90% 71.40%      1.21s  1.90%  runtime.memmove
     1.05s  1.65% 73.04%      3.73s  5.85%  runtime.scanobject
     0.98s  1.54% 74.58%      1.52s  2.38%  runtime.findObject
     0.78s  1.22% 75.80%      3.96s  6.21%  log.(*Logger).formatHeader
     0.74s  1.16% 76.96%     51.16s 80.18%  log.(*Logger).Output
     0.73s  1.14% 78.11%      1.24s  1.94%  runtime.greyobject
     0.58s  0.91% 79.02%     45.08s 70.65%  internal/poll.(*FD).Write
```

- flat：给定函数上运行耗时
- flat%：同上的 CPU 运行耗时总比例
- sum%：给定函数累积使用 CPU 总比例
- cum：当前函数加上它之上的调用运行总耗时
- cum%：同上的 CPU 运行耗时总比例

最后一列为函数名称，在大多数的情况下，我们可以通过这五列得出一个应用程序的运行情况，加以优化 🤔

（2）go tool pprof http://localhost:6060/debug/pprof/heap

```
$ go tool pprof http://localhost:6060/debug/pprof/heap
Fetching profile over HTTP from http://localhost:6060/debug/pprof/heap
Saved profile in C:\Users\fate\pprof\pprof.alloc_objects.alloc_space.inuse_objects.inuse_space.002.pb.gz
Type: inuse_space
Time: Feb 26, 2020 at 10:17pm (CST)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 734.09MB, 100% of 734.09MB total
      flat  flat%   sum%        cum   cum%
  734.09MB   100%   100%   734.09MB   100%  main.Add (inline)
         0     0%   100%   734.09MB   100%  main.main.func1
```

- -inuse_space：分析应用程序的常驻内存占用情况
- -alloc_objects：分析应用程序的内存临时分配情况

（3） go tool pprof http://localhost:6060/debug/pprof/block

（4） go tool pprof http://localhost:6060/debug/pprof/mutex

### 三、PProf可视化界面

这是令人期待的一小节。在这之前，我们需要简单的编写好测试用例来跑一下

#### 编写测试用例

（1）新建 demo_test.go，文件内容：

```go
package main

import "testing"

const url = "Hello World!"

func TestAdd(t *testing.T) {
    s := Add(url)
    if s == "" {
        t.Errorf("Test.Add error!")
    }
}

func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(url)
    }
}
```

（2）执行测试用例

```go
$ go test bench=. -cpuprofile=cpu.prof
goos: windows
goarch: amd64
pkg: demo/gammar
BenchmarkAdd-4           5002905               228 ns/op
PASS
ok      demo/gammar     2.478s
```

-memprofile 也可以了解一下

#### 启动 PProf 可视化界面

**方法一：**

```sh
$ go tool pprof -http=:8080 cpu.prof	
```

**方法二：**

```sh
$ go tool pprof cpu.prof
$ (pprof) web		
```

如果出现`failed to execute dot. Is Graphviz installed? Error: exec: "dot": executable file not found in %PATH%`,就是提示你要安装 `Graphviz` 了(请右拐谷歌)

#### 查看 PProf 可视化界面

（1）Top

![image-20200226222959289](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20200226222959289.png)

(2) Graph

![image-20200226223105986](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20200226223105986.png)

框越大，线越粗代表它占用的时间越大哦

（3）Peek

![image-20200226223142402](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20200226223142402.png)

（4）Source

![image-20200226223225867](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20200226223225867.png)

通过 PProf 的可视化界面，我们能够更方便、更直观的看到 Go 应用程序的调用链、使用情况等，并且在 View 菜单栏中，还支持如上多种方式的切换

你想想，在烦恼不知道什么问题的时候，能用这些辅助工具来检测问题，是不是瞬间效率翻倍了呢 👌

### 四、PProf 火焰图

另一种可视化数据的方法是火焰图，需手动安装原生 PProf 工具：

（1） 安装 PProf

```
$ go get -u github.com/google/pprof
```

（2） 启动 PProf 可视化界面:

```
$ pprof -http=:8080 cpu.prof
```

（3） 查看 PProf 可视化界面

打开 PProf 的可视化界面时，你会明显发现比官方工具链的 PProf 精致一些，并且多了 Flame Graph（火焰图）

它就是本次的目标之一，它的最大优点是动态的。调用顺序由上到下（A -> B -> C -> D），每一块代表一个函数，越大代表占用 CPU 的时间更长。同时它也支持点击块深入进行分析！

![image-20200226223709330](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20200226223709330.png)

# 总结

在本章节，粗略地介绍了 Go 的性能利器 PProf。在特定的场景中，PProf 给定位、剖析问题带了极大的帮助

希望本文对你有所帮助，另外建议能够自己实际操作一遍，最好是可以深入琢磨一下，内含大量的用法、知识点 🤓