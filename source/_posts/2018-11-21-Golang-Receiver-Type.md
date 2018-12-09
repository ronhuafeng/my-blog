---
title: Golang Receiver Type 探索
date: 2018-11-21
categories:
  - 技术记录
  - tech-notes
tags: 
  - golang
  - programming language
---

## 1. 参考

在 Go 的官方 spec 中有以下涉及到类型和方法的章节，如果需要了解具体的细节，可以参考阅读。

- https://golang.google.cn/ref/spec#Method_sets
- https://golang.google.cn/ref/spec#Types
- https://golang.google.cn/ref/spec#Struct_types
- https://golang.google.cn/ref/spec#Composite_literals

核心的概念是 method sets：

> A type may have a method set associated with it. The method set of an interface type is its interface. The method set of any other type T consists of all methods declared with receiver type T. The method set of the corresponding pointer type *T is the set of all methods declared with receiver *T or T (that is, it also contains the method set of T). Further rules apply to structs containing embedded fields, as described in the section on struct types. Any other type has an empty method set. In a method set, each method must have a unique non-blank method name.  
>   
> The method set of a type determines the interfaces that the type implements and the methods that can be called using a receiver of that type.

下面的一些细节基本上都和这段描述相关。

## 2. Duck typing 与方法调用

在很多面向对象的语言中，一个对象都可以“拥有”一些方法，使用例如 `obj.f(a, b, c)` 的形式进行调用。结合语言的类型系统，通过“扩展”、“继承”、“实现”等术语，我们可以将不同的类组织起来。在 Go 语言中采用的是 “duck typing”，没有显式的类型关系定义关键字。当一个类型实现了一个接口的全部方法时，那这个类型就被视为实现了这个接口。

例如：

```go
package main

import "fmt"

type Duck interface {
	Bark()
}

type A struct {}

type B struct {}

func (*A) Bark() {}

func main() {
	var iA interface{} = &A{}
	if _, ok := (iA).(Duck); ok {
		fmt.Println("&A{} is Duck")
	} else {
		fmt.Println("&A{} is not Duck")
	}

	var iB interface{} = &B{}
	if _, ok := (iB).(Duck); ok {
		fmt.Println("&B{} is Duck")
	} else {
		fmt.Println("&B{} is not Duck")
	}
}
```

```
&A{} is Duck
&B{} is not Duck
```

我们可以用原始的类型去调用一个方法，也可以使用一个接口去调用方法。这里就涉及到方法调用者的问题：什么样的对象是一个合法的方法调用者？

至少 `A{}` 不是，因为我们实现 `Duck` 接口的时候，使用的是 `func (*A) Bark()` 进行的定义，而非 `func (A) Bark()`。这样就导致了只有 `A` 类型对象的指针类型才能作为方法调用者去调用 `Bark` 方法。

## 3. 成员函数的参数

在实现中，调用某个类型的成员方法，第一个参数其实是这个方法的实现对象自身，即如果是一个指针的方法，就是这个指针的值，如果是一个对象，就是这个对象的值。

下面使用 Go 1.8.3 展示，因为当前最新的 Go 编译器在打印 stack trace 的时候不再打印函数的参数（这个例子中）。

```go
package main

type R1 struct {
	n int
}

func (r *R1) f(n int) {
	println("received", n)
	panic("just a panic")
}

func main() {
	r := &R1{}
	a := 1
	println(r)
	r.f(a)
}
```

```
0xc420039f70
received 1
panic: just a panic

goroutine 1 [running]:
main.(*R1).f(0xc420039f70, 0x1)
	/Users/bef0rewind/Projects/net example/src/main/receiver_type.go:9 +0xa3
main.main()
	/Users/bef0rewind/Projects/net example/src/main/receiver_type.go:16 +0x5a
```

Stack trace 中函数 `f` 第一个值是指针 `r` 的值。

```
package main

type R1 struct {
	n int
	m int
}

func (r *R1) f(n int) {
	println("received", n)
	panic("just a panic")
}

func (r R1) g(n int) {
	println("received", n)
	panic("just a panic")
}

func main() {
	r := R1{7, 9}
	a := 1
	println(r.n)
	(r).g(a)
}
```

```
7
received 1
panic: just a panic

goroutine 1 [running]:
main.R1.g(0x7, 0x9, 0x1)
	/Users/bef0rewind/Projects/net example/src/receiver_type/main/args.go:15 +0xa3
main.main()
	/Users/bef0rewind/Projects/net example/src/receiver_type/main/args.go:22 +0x58
```

Stack trace 中函数 `g` 第一个值是 `r` 的值 `7` 和 `9`。

从这个实现方式中我们可以推断以下几点：

- Go 语言采用参数传值的方式进行函数调用，因此如果对象很大，使用的对象本身调用函数会带来大量的复制
- 不可能在函数调用中改变函数外的调用者，因为传到函数内部的只是调用者的副本

## 4. 使用接口调用函数

基于这样的成员函数实现方式，我们可以尝试另外一种调用方式：使用接口类型调用一个函数。
这里不是将一个对象转换成特定的接口然后去调用函数，而是使用接口类型本身去进行函数调用。
这种方式在 Go 1.9 中开始支持，在 Go 1.10 开始写入 Go 的 specs。这个例子使用的是 Go master 分支的版本，可能是 Go 1.11。

```go
package main

type M struct {

}

func (*M) f(n int) {
	println("I;m M, with", n)
}

type IM interface {
	f(n int)
}

func main() {
	m := &M{}
	IM.f(m, 7)
}
```

```
I;m M, with 7
```

此外还能使用匿名接口类型去调用函数，例如：

```go
package main

type M struct {

}

func (*M) f(n int) {
	println("I;m M, with", n)
}

func main() {
	m := &M{}
	interface{f(n int)}.f(m, 7)
}
```

运行结果与上面的一段采用 `IM` 接口定义的例子是一样的。

## 5. 注入依赖

有时候一个对象在实例化的时候，它的一些成员方法的行为可能还没有确定，需要依赖外界注入。此时我们可以在对象类型定义中内嵌一个接口，然后在后期传入一个接口的实例来确定其行为。

```go
package main

type BinaryOp interface {
	Compute(a, b int) int
}

type ComputeNode struct {
	x, y int
	BinaryOp
}

func (node *ComputeNode) Result() int {
	return node.BinaryOp.Compute(node.x, node.y)
}

type Add struct {}

func (*Add) Compute(a, b int) int {
	return a + b
}

type Multi struct {}

func (*Multi) Compute(a, b int) int {
	return a * b
}

func main() {
	node := &ComputeNode{x:2, y:3}

	node.BinaryOp = &Add{}
	println(node.Result())

	node.BinaryOp = &Multi{}
	println(node.Result())
}
```

```
5
6
```

注意一定要记得传入接口的实例，在这个例子中如果不给 `node` 传入一个 `BinaryOp` 接口实例，那 `node.BinaryOp` 是 `nil`，在调用 `Compute` 方法的时候就会发生异常。例如将上面的 `main` 函数稍作修改：

```go
func main() {
	node := &ComputeNode{x:2, y:3}

	//node.BinaryOp = &Add{}
	println(node.Result())
}
```

```
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x104d8d7]

goroutine 1 [running]:
main.(*ComputeNode).Result(...)
	/Users/bef0rewind/Projects/net example/src/receiver_type/main/injection.go:13
main.main()
	/Users/bef0rewind/Projects/net example/src/receiver_type/main/injection.go:32 +0x47
```


## 6. 内部机制

内部机制有一些细节。大体就是一个接口 `i` 包含两部分内容（指针），一个是接口代表的方法的集合，一个实现这个接口的具体对象；而一个对象 `obj`，它包含了自己的内存中的值，也能通过其类型获取到 `obj` 实现的方法集合。

将这两个概念记住，在实现一些模式的时候就会少很多心智负担。

## 7. 总结

Go 语言的这套基于 “duck typing” 的机制好不好，争论有很多。不过我一向对这些争论没有特别的倾向，至少理解其机制之后按照其设计思路来用还可以正常使用，而且里面没有复杂的概念和例外情形。

也许我的理解有偏差，但现在还没有发现什么矛盾的地方。
