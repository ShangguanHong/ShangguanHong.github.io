---
title: "Go中的数组与切片"
date: 2022-06-09T21:43:19+08:00
lastmod: 2022-06-09T21:43:19+08:00
draft: false
categories: ["Go学习笔记"]
tags: ["golang"]
---

因为切片 (slice) 比数组更好用，也更安全，Go推荐使用 slice 而不是数组。这次的内容比较了 slice 和数组的区别，也研究了 slice 的一些特有的性质。

## 数组和切片的区别

Go 语言中的切片 (slice) 结构的本质是对数组的封装，他描述了一个数组的片段。无论是数组还是切片，都可以通过下标来访问单个元素。
数组是定长的。

数组是定长的，长度定义好之后，不能再更改。在 Go 语言中，数组是不常见的，因为其长度是类型的一部分，限制了它的表达能力，比如 [3]int 和 [4]int 就是不同的类型。而且切片则非常灵活，它可以动态地扩容，且切片的类型和长度无关。例如：

```go
package main

import (
	"fmt"
)

func main() {
	arr1 := [1]int{1}
	arr2 := [2]int{1, 2}
	if arr1 == arr2 {
		fmt.Println("equal type")
	}
}
```

尝试运行，报编译错误：

```
.\main.go:10:13: invalid operation: arr1 == arr2 (mismatched types [1]int and [2]int)
```

因为两个数组长度不同，根本就不是同一类型，因此不能进行比较。

数组是一片连续的内存，切片实际上是一个结构体，包含三个字段：长度、容量、底层数组，代码结构如下：

```go
// src/runtime/slice.go
type slice struct {
	array unsafe.Pointer // 元素指针
	len   int 			// 长度
	cap   int			// 容量
}
```

需要注意的是，底层数组可以被多个切片同时指向，因此对一个切片的元素进行操作有可能会影响到其他切片。

## 切片如何被截取

截取也是一种比较常见的创建切片的方法，可以从数组或者切片直接截取，需要指定起、止索引位置。

基于已有切片创建新切片对象，被称为 reslice。新切片和老切片共用底层数组，新老切片对底层数组的更改都会影响到彼此。基于数组创建的新切片也是同样的效果：对数组或切片元素做的更改都会影响到彼此。

值得注意的是，新老切片或新切片老数组互相影响的前提是两个共用底层数组，如果因为执行 append 操作使得新切片或老切片底层数组扩容，移动到了新位置，两者就不会相互影响了。所以，问题的关键在于两者是否会共用底层数组。

截取操作采用如下方式：

```go
data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := data[2:4:6] // data[low:high:max]
```

对 data 使用三个索引值，截取出新的 slice。这里 data 可以是数组或者 slice。low 是最低索引值，这里是闭区间，也就是说第一个元素是 data 位于 low 索引处的元素；而 high 和 max 则是开区间，表示最后一个元素只能是索 high-1 处的元素，而最大容量则只能是索引 max-1 处的元素。

要求：max >= high >= low

当 high == low 时，新 slice 为空。

还有一点，high 和 max 必须在老数组或者老 slice 的容量 （cap）范围内。

来看一个例子：运行下面的代码，输出是什么？

```go
package main

import (
	"fmt"
)

func main() {
	slice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s1 := slice[2:5]
	s2 := s1[2:6:7]

	s2 = append(s2, 100)
	s2 = append(s2, 200)

	s1[2] = 20

	fmt.Println(s1)
	fmt.Println(s2)
	fmt.Println(slice)
}
```

运行此段程序，得到如下输出：

```
[2 3 20]
[4 5 6 7 100 200]
[0 1 2 3 20 5 6 7 100 9]
```

得到这个结果的原因是：

s1 从 slice 索引 2（闭区间）到索引 5（开区间，元素真正取到索引 4），长度为 3，容量默认到数组结尾，为 8。

s2 从 s1 的索引 2（闭区间）到索引 6（开区间，元素真正渠道索引 5），容量到索引 7（开区间，真正到索引 6），为 5。slice、s1 和 s2 的初始状态和关系如下图所示。

![image-20220621214058448](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20220621214058448.png)



注意，slice、s1 和 s2 三者的元素指向同一个底层数组。接着，向 s2 尾部追加一个元素 100：

```go
s2 := append(s2, 100)
```

此时，s2 容量刚好够，直接追加。不过这会修改原始数组相对应位置的元素。这一改动，数组和 s1 都可以看得到，现在 slice、s1 和 s2 的初始状态和关系如下图所示。

![image-20220621213945362](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20220621213945362.png)

再次向 s2 追加元素 200：

```go
s2 := append(s2, 200)
```

此时 s2 的容量不够用，需要进行扩容。于是 s2 “另起炉灶”，将原来的元素复制到新的位置，扩大自己的容量。并且为了应对未来可能的 append 带来的再次扩容，s2 会在此次扩容的时候多留一些 buffer，将新的容量扩大为原来的两倍，也就是 10。现在 slice、s1 和 s2 的初始状态和关系如下图所示。

![image-20220621214723033](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20220621214723033.png)

注意，s2 此时的底层数组元素和 slice、s1 已经没有关系了。最后，修改 s1 索引为 2 位置的元素：

```go
s1[2] = 20
```

这次操作只会影响原始数组相对应位置的元素，影响不到 s2 了，它已经 “远走高飞” 了，如下图所示。

![image-20220621215646082](https://raw.githubusercontent.com/ShangguanHong/PictureBed/master/image-20220621215646082.png)

最后执行打印操作，打印 s1 的时候，只会打印出 s1 长度以内的元素。所以只会打印出 3 个元素，虽然他的底层数组不止 3 个元素。

## 切片的容量是怎样增长的

一般都是在想切片追加元素之后，由于容量不足，才会引起扩容。向切片追加元素调用的是 append 函数。append 函数的原型如下：

```go
// src/builtin/builtin.go
func append(slice []Type, elems ...Type) []Type
```

append 函数的参数长度可变，因此可以追加多个值到 slice 中，还可以在切片后面追加 “...” 符号直接传入 slice，即追加切片里所有元素。

```go
slice := append(slice, elem1, elem2)
slice := append(slice, anotherSlice...)
```

append 函数的返回值是一个新的切片，Go 语言的编译器不允许调用了 append 函数后不使用返回值。所以下面的用法是错误的，不能通过编译：

```go
append(slice, elem1, elem2)
append(slice, anotherSlice...)
```

使用 append 函数可以向 slice 追加元素，实际上是往底层数组相应的位置放置要追加的元素。但是底层数组的长度是固定的，如果索引 len-1 所指向的元素已经是底层数组的最后一个元素，那就不能再继续放置新元素了。

这时，slice 会整体迁移到新的位置，并且新底层数组的长度也会增加，使得可以继续放置新增的元素。同时，为了应对未来可能再次发生的 append 操作，新的底层数组的长度，也就是新 slice 的容量需要预留一定的 buffer。否则，每次添加元素的时候，都会发生迁移，成本太高。

新 slice 预留的 buffer 大小是有一定规律的。注意，以下这些说法是不准确的：

说法1：当原 slice 容量小于 1024 的时候，新 slice 容量变成原来的 2 倍；

说法2：当原 slice 容量超过 1024 的时候，新 slice 容量变成原来的 1.25 倍。

为了说明切片的扩容规律，首先通过下面的程序验证一下扩容的行为：

```go
package main

import (
	"fmt"
)

func main() {
	s := make([]int, 0)
	oldCap := cap(s)
	for i := 0; i < 2048; i++ {
		s = append(s, i)
		newCap := cap(s)
		if newCap != oldCap {
			fmt.Printf("[%d->%4d] cap = %-4d  |   after append %-4d   cap = %-4d\n", 0, i-1, oldCap, i, newCap)
			oldCap = newCap
		}
	}
}
```

首先创建一个新的 slice：s，接着，在一个循环里不断地向它 append 新的元素。同时，纪律容量的变化，并且每当容量变化的时候，记录下老的容量，以及添加完元素之后新的容量，并且记下此时向 s 添加的元素。这样就可以观察，新老 s 的容量变化情况，从而找出规律。

代码运行结果如下：

```
[0->  -1] cap = 0     |   after append 0      cap = 1
[0->   0] cap = 1     |   after append 1      cap = 2   
[0->   1] cap = 2     |   after append 2      cap = 4   
[0->   3] cap = 4     |   after append 4      cap = 8   
[0->   7] cap = 8     |   after append 8      cap = 16  
[0->  15] cap = 16    |   after append 16     cap = 32  
[0->  31] cap = 32    |   after append 32     cap = 64  
[0->  63] cap = 64    |   after append 64     cap = 128 
[0-> 127] cap = 128   |   after append 128    cap = 256 
[0-> 255] cap = 256   |   after append 256    cap = 512 
[0-> 511] cap = 512   |   after append 512    cap = 1024
[0->1023] cap = 1024  |   after append 1024   cap = 1280
[0->1279] cap = 1280  |   after append 1280   cap = 1696
[0->1695] cap = 1696  |   after append 1696   cap = 2304
```

在老 s 容量小于 1024 时，新 s 的容量的确是老 s 的 2 倍，目前还算正确。

但当老 s 容量大于等于 1024 的时候，情况就有变化了。例如，向 s 添加元素 1280，老 s 的容量为 1024，新 s 的容量则变成了 1696，两者并不是 1.25 倍的关系 （1696/1024=1.325）；添加完 1696 后，新的容量 2304 当然也不是 1696 的 1.25 倍（2304/1696=1.356）。

要想弄清楚真是的扩容规律是怎么样的，需要深入 Go 语言源码，研究一下扩容函数的具体逻辑。切片的扩容行为本质上是运行时特征，因此 Go 语言的编译器在针对扩容行为时会将其跳转到对应的扩容函数。最简单的寻找入口的方式时对编译后的文件进行逆向工程，这可以通过使用 `go tool compile` 工具添加 `-S` 参数来查看汇编代码。

执行命令：

```
go tool complie -S main.go
```

从实际的汇编代码可以看到，向 s 追加元素的时候，若容量不够，会调用 growslice 函数，所以直接看它的代码：

```go
// src/runtime/slice.go

func growslice(et *_type, old slice, cap int) slice {
	// ...
	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.cap < 1024 {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}

		// ...
		capmem = roundupsize(capmem)
		newcap = int(capmem / et.size)
    	// ...
}
```

如果只看前半部分，现在网络上各种文章里说的 newcap 的规律就是对的。现实是，代码的后半部分还对 newcap 进行了内存对齐，而这个和内存分配策略相关。进行内存对齐之后，新 s 的容量要大于等于老 s 容量的 2 倍或者 1.25 倍。

之后，向 Go 内存管理器申请内存，将老 s 中的数据复制过去，并且将 append 的元素添加到新的底层数组中。最后，向 growslice 函数调用者返回一个新的切片，这个切片的长度并没有变化，而容量却增大了。

再来看一个例子，写出代码的运行结果：

```go
package main

import (
	"fmt"
)

func main() {
	s := []int{5}
	s = append(s, 7)
	s = append(s, 9)
	x := append(s, 11)
	y := append(s, 12)
	fmt.Println(s, x, y)
}
```

直接来一步步地分析上述代码所做的动作，如下表。

| 代码               | 切片对应状态                                                 |
| :----------------- | ------------------------------------------------------------ |
| s := []int{5}      | s 只有一个元素 5， [5]                                       |
| s = append(s, 7)   | s扩容，容量变为2, [5, 7]                                     |
| s = append(s, 9)   | s扩容，容量变为4, [5, 7, 9]。注意，这时 s 长度是 3，只有 3 个元素 |
| x := append(s, 11) | 由于 s 的底层数组仍然有空间，因此并不会发生扩容。这样底层数组就变成了 [5, 7, 9, 11]。注意，此时 s =  [5, 7, 9]，容量为 4；x = [5, 7, 9, 11]，容量为 4。这里 s 不变 |
| y := append(s, 12) | 还是在 s 元素的尾部追加元素，由于 s 的长度为 3，容量为 4，所以直接在底层数组索引为 3 的地方填上 12。结果：s =  [5, 7, 9]，y =  [5, 7, 9, 12]，x =  [5, 7, 9, 12]，x、y 的长度均为 4，容量也均为 4 |

所以最后程序的执行结果是：

```
[5 7 9] [5 7 9 12] [5 7 9 12]
```

这里要注意的是，append 函数执行完后，返回的是一个全新的切片，并且对传入的切片不产生影响。

关于 append 内建函数，最后来看一个例子：

```go
package main

import (
	"fmt"
)

func main() {
	s := []int{1, 2}
	s = append(s, 4, 5, 6)
	fmt.Printf("len = %d, cap = %d", len(s), cap(s))
}
```

代码运行结果是：

```
len = 5, cap = 6
```

如果按照 “原 s 长度小于 1024 的时候，扩容后容量增加 1 倍。添加元素  4 的时候，容量变为 4；添加元素 5 的时候容量不变；添加元素 6 的时候容量增加 1 倍，变成 8”，那么运行结果应该是：

```
len = 5, cap = 8
```

这显然是错误的，再来仔细看看源代码：

```go
// src/runtime/slice.go

func growslice(et *_type, old slice, cap int) slice {
	// ...
	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.cap < 1024 {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}

		// ...
		capmem = roundupsize(capmem)
		newcap = int(capmem / et.size)
    	// ...
}
```

这个函数的入参分别是：元素的类型、老 slice、新 slice 要求的最小容量。

例子中 s 原来只有 2 个元素，len 和 cap 都为 2，append 了 3 个元素后，长度变为 5，容量最小要变成 5。当调用 growslice 函数是，传入的第 3 个参数应该为 5。即 cap=5。而另一方面，doublecap 是原 slice 容量的 2 倍，等于 4.满足第一个 if 条件，所以 newcap 变成了 5。

接着调用了 roundupsize 函数，传入 40。因为 capmem 会被赋值成 40：

```go
capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
```

再来看内存对齐相关的 roundupsize 函数的代码：

```go
// src/runtime/msize.go

const (
	_MaxSmallSize   = 32768
	smallSizeDiv    = 8
	smallSizeMax    = 1024
	largeSizeDiv    = 128
	_NumSizeClasses = 68
	_PageShift      = 13
)

func roundupsize(size uintptr) uintptr {
	if size < _MaxSmallSize {
		if size <= smallSizeMax-8 {
			return uintptr(class_to_size[size_to_class8[divRoundUp(size, smallSizeDiv)]])
		} else {
			return uintptr(class_to_size[size_to_class128[divRoundUp(size-smallSizeMax, largeSizeDiv)]])
		}
	}
	if size+_PageSize < size {
		return size
	}
	return alignUp(size, _PageSize)
}
```

其中 divRoundUp 函数就是一个向上取整函数，其源码如下：

```go
// src/runtime/stubs.go

// divRoundUp returns ceil(n / a).
func divRoundUp(n, a uintptr) uintptr {
	// a is generally a power of two. This will get inlined and
	// the compiler will optimize the division.
	return (n + a - 1) / a
}
```



最终将返回这个式子的结果：

```go
class_to_size[size_to_class8[divRoundUp(size, smallSizeDiv)]]
```

这是 Go 源码中有关内存分配的两个 slice。class_to_size 通过 spanClass 获取 span 划分的 object 大小，而 size_to_class8 通过 size 获取它的 spanClass。

```go
var class_to_size = [_NumSizeClasses]uint16{0, 8, 16, 24, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 352, 384, 416, 448, 480, 512, 576, 640, 704, 768, 896, 1024, 1152, 1280, 1408, 1536, 1792, 2048, 2304, 2688, 3072, 3200, 3456, 4096, 4864, 5376, 6144, 6528, 6784, 6912, 8192, 9472, 9728, 10240, 10880, 12288, 13568, 14336, 16384, 18432, 19072, 20480, 21760, 24576, 27264, 28672, 32768}

var size_to_class8 = [smallSizeMax/smallSizeDiv + 1]uint8{0, 1, 2, 3, 4, 5, 5, 6, 6, 7, 7, 8, 8, 9, 9, 10, 10, 11, 11, 12, 12, 13, 13, 14, 14, 15, 15, 16, 16, 17, 17, 18, 18, 19, 19, 19, 19, 20, 20, 20, 20, 21, 21, 21, 21, 22, 22, 22, 22, 23, 23, 23, 23, 24, 24, 24, 24, 25, 25, 25, 25, 26, 26, 26, 26, 27, 27, 27, 27, 27, 27, 27, 27, 28, 28, 28, 28, 28, 28, 28, 28, 29, 29, 29, 29, 29, 29, 29, 29, 30, 30, 30, 30, 30, 30, 30, 30, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32}
```

传进去的 size 等于 40，所以 (divRoundUp(size, smallSizeDiv)]]) = 5；获取 size_to_class8 数组中索引为 5 的元素为 4；获取 class_to_size 中索引为 4 的元素为48。

最终，新的 slice 的容量为 6：

```go
newcap = int(capmem / ptrSize) // 48/8=6
```

最后一个问题是，可以直接向一个 nil 的切片添加元素吗？会发生什么？

答案是可以的。其实向 nil 切片或者空切片执行 append 操作时，都会调用 growslice 函数来使底层数组进行扩容。最终都是调用 mallocgc 来向 Go 的内存管理器申请一块内存，然后再赋值给原来的 nil 切片或者空切片。

## 切片作为函数参数会被改变吗

前面说到，切片其实就是一个结构体，包含了三个成员：len，cap，arrry，分别表示切片长度，容量，底层数组的地址。

在 slice 作为函数参数时，就是一个普通的结构体。从这个角度其实很好理解：若直接传 slice，在调用者看来，实参 slice 并不会被函数中对形参的操作改变，形参是实参的一个复制；若传入的是 slice 的指针，则会影响实参。

需要注意，不论传入的是 slice 还是 slice 指针，如果改变了 slice 底层数组的数据，都会反应到实参 slice 的底层数组。为什么会改变底层数组的数据呢？因为底层数组在 slice 结构体里是一个指针，尽管 slice 结构体自身不会被改变，也就是说底层数组的地址不会被改变，但是通过指向底层数组的指针，当然可以改变切片的底层数组的数据。

通过 slice 的 array 字段就可以拿到数组的地址。在代码里，可以直接通过类似 s[i] = 10 这种操作改变 slice 底层数组元素值。

另外，需要说明的是，Go 语言中的函数参数传递，只有值传递，没有引用传递。

来看一个代码片段：

```go
package main

import (
	"fmt"
)

func main() {
	s := []int{1, 1, 1}
	f(s)
	fmt.Println(s)
}

func f(s []int) {
	for i := range s {
		s[i] += 1
	}
}
```

程序输出结果是：

```
[2 2 2]
```

确实改变了原始 slice 的底层数据。这里向 f 函数传递的是一个 slice 的副本，在 f 函数中，s 只是 main 函数中 s 的一个复制。在 f 函数内部，对 s 的作用并不会改变外层 main 函数的 s。注意，这里的不会改变指的是 slice 结构体。

要想改变外层 slice 结构体，只有将返回的新 slice 赋值到原始 slice 中，或者向函数传递一个指向 slice 的指针。再来看一个例子：

```go
package main

import (
	"fmt"
)

func myAppend(s []int) []int {
	// 这里 s 结构体虽然改变了，但并不会改变外层的 s 结构体
	s = append(s, 100)
	return s
}

func myAppendPtr(s *[]int) {
	// 会改变外层 s 结构体本身
	*s = append(*s, 100)
}

func main() {
	s := []int{1, 1, 1}
	newS := myAppend(s)

	fmt.Println(s)
	fmt.Println(newS)

	s = newS
	myAppendPtr(&s)
	fmt.Println(s)
}
```

运行结果：

```
[1 1 1]
[1 1 1 100]
[1 1 1 100 100]
```

在 myAppend 函数里，虽然改变了 s，但它只是一个值传递，并不会影响外层的 s 结构体，也就是说长度、容量、底层数组的地址都不会变，因此第一行打印出来的结果仍然是 [1 1 1]。

而 newS 是一个新的切片，它是基于 s 得到的，长度增加了 1，容量变成了 s 的两倍。因此打印的是追加了 100 之后的结果：[1 1 1 100]。

之后，将 newS 赋值给了 s，s 这时变成了一个新的切片。最后，再给 myAppendPtr 函数传入了一个 s 指针，这次它真的被改变了：[1 1 1 100 100]。

## 内建函数 make 和 new 的区别是什么

首先 make 和 new 均是 Go 语言的内置的用来分配内存的函数。但适用的类型不同，前者适用于 slice、map、channel 等引用类型；后者适用于 int 型、数组、结构体等值类型。

其实，两者的函数形式及调用形式不同，函数形式如下：

```go
func make(t Type, size ...IntegerType) Type
func new(Type) *Type
```

前者返回的是一个值，后者返回一个指针。

使用上，make 返回初始化之后的类型的引用，new 会为类型的新值分配已置零的内存空间，并返回指针。例如：

```go
s := make([]int, 0, 10) // 使用 make 创建一个长度为 0，容量为 10 的切片

a := new(int) // 使用 new 分配一个零值的 int 型
*a = 5
```

【思考一下】类型于 make 函数的声明里参数类型前的三个点：“...” 有什么作用呢？

“...”表示可以传入一个或多个实参，这使得函数调用更加灵活。例如：

```go
func append(slice []Type, elems ...Type) []Type
```

Go 内置的 append 函数向切片中追加元素，elems 参数是 “...” 的形式。因此调用 append 函数时比较灵活，既可以追加单个元素，也可以追加多个元素，还可以追加一个切片：

```go
package main

import (
	"fmt"
)

func main() {
	s := make([]int, 0)
	fmt.Println(s) // s = []

	s = append(s, 1) // 追加单个元素
	fmt.Println(s)   // s = [1]

	s = append(s, []int{2, 3, 4}...) // 追加一个切片
	fmt.Println(s)                   // s = [1 2 3 4]

	s = append(s, 5, 6, 7) // 追加多个元素
	fmt.Println(s)         // s = [1 2 3 4 5 6 7]
}
```

运行结果：

```
[1 1 1]
[1 1 1 100]
[1 1 1 100 100]
```



