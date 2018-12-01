---
title: Escape from escape analysis
tags:
  - golang
  - programming language
categories:
  - 技术记录
  - tech-notes
date: 2018-11-30 00:00:00
---


## 1. 逃逸分析背景

Go 语言采用了并发的（Concurrent）、非移动的（Non-Movable）、非分代的（Non-Generational）、基于三色（Tri-color）标记的垃圾回收（Garbage Collection）算法，只在 特定阶段开启写屏障（write barrier）。
特点是全局停顿时间比较少，在一些场景下是十微秒级别的。

垃圾回收算法针对的是堆（heap）中的内存。
为了减少垃圾回收的时间消耗，Go 语言在编译阶段通过静态分析算法对程序的结构进行分析，尽可能讲对象分配在栈上（如果这个对象的生命周期在它定义的函数返回时就结束的话）。
这一算法也利用了 Go 语言在函数传递参数时总是传递参数的值这一个语言特性。

而静态分析不总是完备的，会有一些本来可以分配在栈上的对象被 Go 的编译器分配在了堆上。
如这篇文章《[Golang escape analysis](http://www.agardner.me/golang/garbage/collection/gc/escape/analysis/2015/10/18/go-escape-analysis.html)》所描述的一些例子一样，有些对象本来可以避免逃逸（Escape，指的是对象被分配在堆上）。

对于某些场景，我们确定一个对象肯定可以（也应当）被分配在栈上，但是它却逃逸了。
这样在某些关键路径上的逃逸的对象会造成大量的分配和垃圾回收。

## 2. Go 版本

使用的 Go 版本为今晚刚从 master 分支上 pull 下的源码直接构建。

```
ThinkPad-X1-Carbon:bin bef0rewind$ ./go version
go version devel +42e8b9c3a4 Fri Nov 30 15:17:34 2018 +0000 darwin/amd64
```

## 3. 示例

```go
// file: escape.go
package main

import "fmt"

type BigTempObject struct {
	/// ...
	field1 int
}

func causeEscape(i interface{}) {
	switch i.(type) {
	case *BigTempObject:
		println(i)
	default:
		fmt.Println(i)
	}
}

func main() {
	obj := BigTempObject{}
	addrObj := &obj

	causeEscape(addrObj)
}
```

使用 `go run -gcflags="-m -m" escape.go` 可以在运行时输出逃逸分析的结果。

```
./escape.go:10: cannot inline causeEscape: unhandled op TYPESW
./escape.go:19: cannot inline main: non-leaf function
./escape.go:10: leaking param: i
./escape.go:10:         from ... argument (arg to ...) at ./escape.go:15
./escape.go:10:         from *(... argument) (indirection) at ./escape.go:15
./escape.go:10:         from ... argument (passed to call[argument content escapes]) at ./escape.go:15
./escape.go:15: causeEscape ... argument does not escape
./escape.go:23: addrObj escapes to heap
./escape.go:23:         from addrObj (passed to call[argument escapes]) at ./escape.go:23
./escape.go:21: &obj escapes to heap
./escape.go:21:         from addrObj (assigned) at ./escape.go:21
./escape.go:21:         from addrObj (interface-converted) at ./escape.go:23
./escape.go:21:         from addrObj (passed to call[argument escapes]) at ./escape.go:23
./escape.go:20: moved to heap: obj
(0x10904e0,0xc420080050)
```

`obj` 可以分配在栈上，因为在 `main` 函数返回时（栈退出），这个变量占用的空间就可以安全被用在其他地方了。
但是 “./escape.go:20: moved to heap: obj” 说明 `obj` 被分配在了堆上。

## 4. 小技巧

如何改变这个分析结果，需要一点小技巧。

关键词是 `uintptr` 类型。
Go 语言中对 `uintptr` 是这样描述的：

> uintptr is an integer type that is large enough to hold the bit pattern of any pointer.

比如在 64-bit Linux 系统上 `uintptr` 被定义成为了 `uint64`。
Go 中合法的类型转换为：`normal pointer` ⟷ `unsafe.Pointer` ⟷ `uintptr` 。
因此我们可以把上面的程序中的 `addrObj` 转换为 `uintptr`。
这样 Go 编译器不再认为 `addrObj` 同后面函数 `causeEscape` 使用的参数 `i` 存在引用关系，从而绕过 Escape Analysis Algorithm 。
为了防止垃圾回收过程中 `obj` 被回收，可以使用 `obj.field1 = 0` 来保持 `obj` 活跃。

修改后的代码如下：

```go
package main

import (
	"fmt"
	"unsafe"
)

type BigTempObject struct {
	/// ...
	field1 int
}

func causeEscape(i interface{}) {
	switch i.(type) {
	case *BigTempObject:
		println(i)
	default:
		fmt.Println(i)
	}
}

func main() {
	obj := BigTempObject{}
	addrObj := &obj
	intAddr := uintptr(unsafe.Pointer(addrObj))
	causeEscape((*BigTempObject)(unsafe.Pointer(intAddr)))
	obj.field1 = 0
}
```

使用 `go run -gcflags="-m -m" escape.go` 运行结果：

```
./escape.go:13: cannot inline causeEscape: unhandled op TYPESW
./escape.go:22: cannot inline main: non-leaf function
./escape.go:13: leaking param: i
./escape.go:13:         from ... argument (arg to ...) at ./escape.go:18
./escape.go:13:         from *(... argument) (indirection) at ./escape.go:18
./escape.go:13:         from ... argument (passed to call[argument content escapes]) at ./escape.go:18
./escape.go:18: causeEscape ... argument does not escape
./escape.go:26: (*BigTempObject)(unsafe.Pointer(intAddr)) escapes to heap
./escape.go:26:         from (*BigTempObject)(unsafe.Pointer(intAddr)) (passed to call[argument escapes]) at ./escape.go:26
./escape.go:24: main &obj does not escape
(0x10904e0,0xc42003bf70)
```

可以看到 `obj` 不再逃逸，主要是 `intAddr` 中断了逃逸分析算法构建的指针依赖关系（表示为一个有向图）。

## 5. 一点感想

我们可以做到不代表一定去做，有风险也不代表禁区，采取什么样的行动是个人权衡后的选择。  
什么原因导致了人们做了不同的选择，而人们不同的选择又导致了什么结果？  
多样性是这个世界的现状，黑暗面与光明面同在。
May the force be with you.  