title: defer, panic and recover in Golang
date: 2018-08-21
categories:
  - 技术记录
  - tech-notes
tags: golang
---

## 1. 什么是异常处理

程序在执行过程中有可能出现异常状态，比如获取一个不再有效指针指向的内容、除零等。
一般语言都提供了异常处理机制来应对这些情形，例如 Java 的 `try`/`catch`/`finally` 机制（https://docs.oracle.com/javase/tutorial/essential/exceptions/catch.html）、
Python 的 `try`/`raise`/`except`/`finally` 机制（https://docs.python.org/3/tutorial/errors.html）等。

## 2. Go 语言中的异常处理机制

Go 语言中使用的是 `defer`/`panic`/`recover` 机制来处理异常。Go 语言官方博客的《[Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover)》讲述了这个机制的具体应用方式。

还有一些其他教程对这个机制的使用方法、适用场景进行了进一步阐述：

- The Right Places To Call The recover Function：https://go101.org/article/panic-and-recover-more.html ，几种使用 `recover` 恢复 Go 语言中 `panic` 的适用场景
- 7.异常处理：https://www.kancloud.cn/liupengjie/go/578555 ，看云发布的 Go 语言教程中涉及到异常处理的章节，里面涉及到很多使用示例细节

如果搜索 “golang 异常处理”，类似的教程有很多。里面的核心思想大体就是：用 `defer` + `recover`  处理一个 `panic`，`defer` 结构要在 `panic` 触发之前被定义而且 `recover` 要直接在在 `defer` 结构定义的函数中被调用（而不是被直接调用或者在函数内部的其他函数中被调用）。

## 3. `defer` 语法糖的部分原理

在讲述 `defer` 机制的文章中，都会提到一个函数中多个 `defer` 结构执行的顺序和定义顺序是相反的，即后定义的 `defer` 结构总是先被执行。为什么会出现这样的情况？例如下面的代码：

```go
func g(n int) {
  println(n)
}

func h(str string) {
  println(str)
}

func f() {
  defer g(0)
  defer h("h")
}
```

调用 `f` 输出为：

```
h
0
```

常见的函数调用流程为：

- 将函数使用的参数压入栈
- 执行函数指令
- 函数执行结束返回到调用点

如果 `defer` 相关的代码也是这么执行的话，那么为什么不是： `0` 入栈 - 执行 `g` - `g` 返回 - `"h"` 入栈 - 执行 `h` - `h` 返回 这个顺序呢？
按照这个顺序执行，调用 `f` 输出应该是 `0` 在 `h` 前面符合预期。是不是 Go 语言中执行 `defer` 时采用了特殊的处理流程？

是，也不是。

太阳底下无新鲜事，`defer` 不过是一个语法糖，用来对一个函数 `deferproc` 进行包装。

```go
// Create a new deferred function fn with siz bytes of arguments.
// The compiler turns a defer statement into a call to this.
//go:nosplit
func deferproc(siz int32, fn *funcval)
```

`deferproc` 创建一个延迟调用的函数，其参数为 `siz` （延迟调用的函数的参数占用的字节数量）和 `fn`（被延迟调用的函数本身）。
当 Go 程序的编译器遇到 `defer f()`，会将这条语句翻译为一条 `deferproc` 和一条 `deferreturn` 。
其中 `deferproc` 把被调用的函数及其参数挂载在 goroutine （Go 中的并发单元，协程）结构的一个链表上；
`deferreturn` 从链表上取下一个挂载的被延迟执行的函数，执行它。

如何使用技巧绕过 `defer` 关键字，模拟类似效果？
可以使用 `linkname` 方法来把 Go 语言运行时的一些关键函数导出，从而进行某些不常见的操作。

```go
package main

import (
	_ "runtime"
	"unsafe"
)

type Eface struct {
	_type uintptr
	Data  unsafe.Pointer
}

func EfaceOf(ep *interface{}) *Eface {
	return (*Eface)(unsafe.Pointer(ep))
}

type Funcval struct {
	fn uintptr
	// variable-size, fn-specific data here
}

//go:linkname Deferproc runtime.deferproc
func Deferproc(siz int32, fn *Funcval)

//go:linkname Deferreturn runtime.deferreturn
func Deferreturn(arg0 uintptr)

func main() {
	var f = func() {
		println("hacked defer")
	}
	var fI interface{} = f

	// Attach a defer struct to the current goroutine struct
	Deferproc(0, (*Funcval)(EfaceOf(&fI).Data))

	defer func() {
		println("original defer")
	}()

	// Run a deferred function if there is one
	Deferreturn(0)
}
```

这段代码会输出：

```output
original defer
hacked defer
```

当然，如果是使用 `defer` 关键字，Go 语言的编译器会选择合适的位置插入 `deferreturn` 语句，而不是像上述代码中一样手动放在结束位置处。

## 4. `recover` 生效位置的设计原因推测

言归正传，`panic` 发生后，会根据函数调用顺序逐层上报，直到最后一层被抛出到系统导致崩溃或者被 `recover` 机制处理。
那么如果被 `recover` 处理，这个过程是怎么生效的？

很多教程中都提到 `recover` 一定要在 `defer` 声明的函数里面（既不是这个函数本身也不能是函数里面的其他函数里面）才能正确处理当前的 `panic` 。

```go
// case 1, not work
defer recover()

// case 2, not work
defer func() {
  func() {
    recover()
  }
}()

// case 3, work
defer func() {
  recover()
}()
```

为什么呢？

先不考虑实现，先从理念上分析一下。

1. `defer` 直接作用于 `recover()`：无法根据 `recover()` 的返回值来进行不同类型的 `panic` 处理
2. 在被 `defer` 作用的函数内部的函数 `g` 中使用 `recover()`：如果 `g` 是一个第三方库的函数，无法保证其中没有未知的 `recover` 意外处理了系统中的 `panic`。

因此事实上也只能通过这样的约束来使这个异常处理机制看上去直观易处理一些。当然通过对 Go 编译器进行修改，还是有办法使得上面三种情况下 `recover` 都可以中断 `panic` 向上层传递过程的。

此外，由于被 `defer` 处理的函数被挂载在 goroutine 结构的一个链表上，因此当 `panic` 发生时，可以直接从这个链表上取下被延迟执行的函数一个个执行。
这也是 `recover` 要放在 `deferred function` 中的原因，因为这些函数是肯定可以执行到的。

## 5. 总结

不能说 Go 中这个异常处理机制有多高明，基本上属于现代语言标配。了解更多背后的原理，在使用时可以更坚定一些。

此外，最近看到一本书《最好的告别》（https://book.douban.com/subject/26576861/）。

![Being Mortal](https://images-na.ssl-images-amazon.com/images/I/41rwxKTGwXL._SX308_BO1,204,203,200_.jpg)

豆瓣上的介绍：

> 当独立、自助的生活不能再维持时，我们该怎么办？在生命临近终点的时刻，我们该和医生谈些什么？应该如何优雅地跨越生命的终点？对于这些问题，大多数人缺少清晰的观念，而只是把命运交由医学、技术和陌生人来掌控。影响世界的医生阿图•葛文德结合其多年的外科医生经验与流畅的文笔，讲述了一个个伤感而发人深省的故事，对在21世纪变老意味着什么进行了清醒、深入的探索。

`defer` / `finally` 这些关键字让我们可以控制函数退出时的行为，但是我们自身呢？也许考虑这些问题可以让我们自身活得有意义一些。

推荐大家看一下。
