---
title: Golang 中学到的新东西
categories:
  - 技术记录
date: 2018-01-14 11:53:30
tags:
  - Golang
---

# 数据类型

## 1. `string` 类型

`string` 类型使用 2 个 word（64 bit 系统为 8 byte * 2）表示：一个 word 是指针，指向字符串存储区域；一个 word 表示长度数据。

## 2. `slice` $\leftrightarrow$ `unsafe.Pointer`

```go
s := make([]byte, 200)
ptr := unsafe.Pointer(&s[0])
```

```go
var ptr unsafe.Pointer
s := ((*[1<<10]byte)(ptr))[:200]
```
or
```go
var ptr unsafe.Pointer
var s1 = struct {
    addr uintptr
    len  int
    cap  int
}{ptr, length, length}

s := *(*[]byte)(unsafe.Pointer(&s1))
```
or
```go
var o []byte
sliceHeader := (*reflect.SliceHeader)((unsafe.Pointer(&o)))
sliceHeader.Cap = length
sliceHeader.Len = length
sliceHeader.Data = uintptr(ptr)
```

## 3. `map` 实现

整个页面的内容对我来说都是新的：https://tiancaiamao.gitbooks.io/go-internals/content/zh/02.3.html
不过这个页面描述的内容和最新的 Golang source 有一定差别。

读 `HashMap` 的实现，里面的一些核心关键词：`bucket`、`overflow` 让我理解起来有些困难。查询 `HashMap` 相关的一些资料后有了进一步了解。

1. `bucket` 一般使用某种 `array` 管理，从 `key` 经过 `hash-function` 映射的 `hash-value`（可能截取一部分，也可以视作 sub-hash，我自己编的）作为 `index` 直接得到。一个 `bucket` 中可能包含多个不同的 `hash-value` ，它们截取那一部分得到的 `index` 相同。因此 `bucket` 会用一个数据结构管理这些冲突的值，可能是 `linked-list` 或者 `tree-map` 之类的。这些内部的数据结构中的 `node` 存储着真正对应 `map` 的 `key\value` 对(pair)。
2. 如果 `bucket` 太满了，比如元素的数量超过 `bucket` 数量一定倍数（`load factor`），则会进行扩容，所有元素都被 `rehashed` 到一个新的值。
3. 采用这种方式实现 `HashMap`，`bucket` 可以有两种选择：
  - **Direct chaining** 只存一个指向冲突元素集合的 `header` 
  - **Seperate Chaining** 在 `bucket` 存一部分（一个）元素集合（Golang `HashMap` 实现里放了 8 个），和一个指向剩下冲突元素集合的 `header`
4. 上面 `header` 指向的元素集合叫 `overflow list` 或者 `overflow some-other-data-structure`

有了这些背景后，看代码应该会比较清晰了。

目前 Golang 中的 `bucket` 是为 `insert` 操作优化的，找到第一个空余位置就可以插入，但是删除的时候要把所有相同 `key` 的元素都删掉，要遍历 `bucket` 的 `overflow` 集合。

如果 key 或者 value 小于 128 字节，那么它们是直接在 `bucket` 存储值，否则存指向数据的指针。

## 4. `nil` 语义

按照 Golang 规范，任何类型在未初始化时都对应一个零值：
- `bool` $\rightarrow$ `true`
- `integer` $\rightarrow$ `0`
- `string` $\rightarrow$ `""`
- `pointer`/`function`/`interface`/`slice`/`channel`/`map` $\rightarrow$ `nil`

### 关于 `interface{}`

```go
var v *T           // v == nil
var i interface{}  // i == nil
i = v              // i != nil
```

### 关于 `channel`

一些操作规则：
- 读写一个 `nil` 的 `channel` 会立即阻塞
- 读一个关闭的 `channel` 会立刻返回一个 `channel` 元素类型的零值，即 `chan int` 会返回 `0`
- 写一个关闭的 `channel` 会导致 `panic`


## 5. 函数调用

### 汇编

可以看一下这个 Golang 的官方介绍页面：https://golang.org/doc/asm

**add.go**
```go
package add

func Add(a, b uint64) uint64
```

**add_amd64.s** 或使用其他平台后缀，和 **add.go** 在同一个目录
```nasm
TEXT    ·Add+0(SB),$0-24
MOVQ    a+0(FP),BX
MOVQ    b+8(FP),BP
ADDQ    BP,BX
MOVQ    BX,res+16(FP)
RET     ,
```

### Golang 调用 C

**add.c** ，和 **add.go** 在同一个目录
```c
void ·Add(uint64 a, uint64 b, uint64 ret) {
    ret = a + b;
    FLUSH(&ret);
}
```
编译这个包：`go install add`

C 文件中需要包含 `runtime.h` 头文件。因为 Golang 使用特殊寄存器存放像全局 `struct G` 和 `struct M` ，包含这个文件可以让所有链接到 Go 的 C 文件感知这一点，避免编译器使用这些特定的寄存器做其他用途。

上面示例中返回值为空，使用 `ret` 作为返回值，`FLUSH` 在 `pkg/runtime/runtime.h` 中定义为 `USED()` ，防止编译器优化掉对某个变量的赋值操作（因为看不到这个变量在后面其他地方使用了）。

### 函数调用时的内存布局

Golang 中使用的 C 编译器是 plan9 的 C 编译器，与 gcc 有一定差异。
这个页面中有部分基础介绍：
https://tiancaiamao.gitbooks.io/go-internals/content/zh/03.1.html

如果返回多个值，`func f(a, b int) (d, e int)` 内存布局如下所示：
```
slot for e
slot for d
b
a 
<- SP
```
调用后为
```
slot for e
slot for d
b
a <- FP
PC <- SP
f's stack
```

plan9 的 C 汇编器对被调用函数的参数值的修改是会返回到调用函数中的。
```nasm
MOVQ    BX,d+16(FP)
...
MOVQ    BX,e+24(FP)
```

## 6. `go` 关键字

`f(1, 2, 3)` 的汇编:
```nasm
MOVL    $1, 0(SP)
MOVL    $2, 4(SP)
MOVL    $3, 8(SP)
CALL    f(SB)
```

`go f(1, 2, 3)` 的汇编：
```nasm
MOVL    $1, 0(SP)
MOVL    $2, 4(SP)
MOVL    $3, 8(SP)
PUSHQ   $f(SB)
PUSHQ   $12
CALL    runtime.newproc(SB)
POPQ    AX
POPQ    AX
```
`12` 是参数占用的大小，`runtime.newproc` 函数接受的参数为：参数大小、新的 goroutine 要运行的函数、函数的参数。`runtime.newproc` 会新建一个栈空间，将栈参数的 12 个字节复制到新的栈空间，并让栈指针指向参数。可以看做 `runtime.newproc(size, f, args)` 。

## 7. `defer` 关键字

`return x` 不是原子语句，函数执行顺序为：
1. 给返回值赋值
2. `defer` 调用
3. `return`

`defer` 实现对应 `runtime.deferproc`，其出现的地方插入指令 `call runtime.deferproc` ，函数返回之前的地方，插入 `call runtime.deferreturn` 。 goroutine 的控制结构中有一张表记录 `defer`，表以栈行为运作。


# 引用

1. https://tiancaiamao.gitbooks.io/go-internals
2. http://gki.informatik.uni-freiburg.de/teaching/ss11/theoryI/07_Hashing_Chaining.pdf