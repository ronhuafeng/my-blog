---
title: Golang 系统调用/Syscall
categories:
  - 技术记录
date: 2018-01-15 20:59:11
tags:
  - Golang
---
# 概述

很多和系统相关的函数都需要调用系统 API，例如读写文件的函数。Golang 对一些系统调用接口进行了封装，提供了 Golang 函数让用户调用，例如：
```go
func Read(fd int, p []byte) (n int, err error)
func Write(fd int, p []byte) (n int, err error)
```
同时，Golang 也提供了对 Syscall 的直接调用支持：
```go
func Syscall(trap, a1, a2, a3 uintptr) (r1, r2 uintptr, err Errno)
func Syscall6(trap, a1, a2, a3, a4, a5, a6 uintptr) (r1, r2 uintptr, err Errno)

func RawSyscall(trap, a1, a2, a3 uintptr) (r1, r2 uintptr, err Errno)
func RawSyscall6(trap, a1, a2, a3, a4, a5, a6 uintptr) (r1, r2 uintptr, err Errno)
```
`RawSyscall` 和 `RawSyscall6` 是对操作系统 Syscall 的直接调用；`Syscall` 和 `Syscall6` 会在调用操作系统 Syscall 前调用 `runtime·entersyscall` ，在操作系统 Syscall 返回后调用 `runtime·exitsyscall` 。

这四个函数都是使用汇编语言实现，代码和具体的硬件体系结构和操作系统有关。`RawSyscall` 和 `RawSyscall6` 的行为和 C 语言中系统调用很类似，在这里不展开描述。而 `Syscall` 和 `Syscall6` 的行为（在进行真正的系统调用前后插入额外操作）与 Golang 的调度器关系紧密，在下面会进行要点描述。

`Syscall` 的关键在于 `runtime·entersyscall` 和 `runtime·exitsyscall` ，而 `runtime·entersyscall` 还有一个行为有部分差异的版本 `runtime·entersyscallblock` 。

# `runtime·entersyscall`

```go
func entersyscall(dummy int32) {
	reentersyscall(getcallerpc(), getcallersp(unsafe.Pointer(&dummy)))
}
```

```go
func reentersyscall(pc, sp uintptr) {
	_g_ := getg()

	// Disable preemption because during this function g is in Gsyscall status,
	// but can have inconsistent g->sched, do not let GC observe it.
	_g_.m.locks++

	// Entersyscall must not call any function that might split/grow the stack.
	// (See details in comment above.)
	// Catch calls that might, by replacing the stack guard with something that
	// will trip any stack check and leaving a flag to tell newstack to die.
	_g_.stackguard0 = stackPreempt
	_g_.throwsplit = true

	// Leave SP around for GC and traceback.
	save(pc, sp)
	_g_.syscallsp = sp
	_g_.syscallpc = pc
	casgstatus(_g_, _Grunning, _Gsyscall)
	if _g_.syscallsp < _g_.stack.lo || _g_.stack.hi < _g_.syscallsp {
		systemstack(func() {
			print("entersyscall inconsistent ", hex(_g_.syscallsp), " [", hex(_g_.stack.lo), ",", hex(_g_.stack.hi), "]\n")
			throw("entersyscall")
		})
	}

	if trace.enabled {
		systemstack(traceGoSysCall)
		// systemstack itself clobbers g.sched.{pc,sp} and we might
		// need them later when the G is genuinely blocked in a
		// syscall
		save(pc, sp)
	}

	if atomic.Load(&sched.sysmonwait) != 0 {
		systemstack(entersyscall_sysmon)
		save(pc, sp)
	}

	if _g_.m.p.ptr().runSafePointFn != 0 {
		// runSafePointFn may stack split if run on this stack
		systemstack(runSafePointFn)
		save(pc, sp)
	}

	_g_.m.syscalltick = _g_.m.p.ptr().syscalltick
	_g_.sysblocktraced = true
	_g_.m.mcache = nil
	_g_.m.p.ptr().m = 0
	atomic.Store(&_g_.m.p.ptr().status, _Psyscall)
	if sched.gcwaiting != 0 {
		systemstack(entersyscall_gcwait)
		save(pc, sp)
	}

	// Goroutines must not split stacks in Gsyscall status (it would corrupt g->sched).
	// We set _StackGuard to StackPreempt so that first split stack check calls morestack.
	// Morestack detects this case and throws.
	_g_.stackguard0 = stackPreempt
	_g_.m.locks--
}
```

`runtime·entersyscall` 主要完成以下几件事：
1. 声明函数为 `NOSPLIT` ，不触发栈扩展检查
2. 禁止抢占
3. 通过 `dummy` 参数获得调用者的 `SP` 和 `PC` 的值，并保存到 goroutine 的 `syscallsp` 和 `syscallpc` 字段。同时记录 `syscallstack` 和 `syscallguard` ，使得垃圾回收器只对系统调用前的栈进行 **mark-sweep** （cgo 机制也利用了 `entersyscall` 来使得 cgo 中运行的代码不受垃圾回收机制管理）。
4. 将 goroutine 的状态切换到 Gsyscall 状态。
5. 唤醒后台线程 `sysmon` （这个线程会监控执行 `syscall` 的线程，如果超过某个时间阈值，就会将 M 与对应的 P 解除绑定）。
6. 置空 M 的 `mcache` 、将 P 的 `m` 字段，切换 P 的状态到 `Psyscall` 
7. 检查是否需要垃圾回收
8. 通过 `g->stackguard0 = StackPreempt` 使得出现 *split stack* 时可以通过 `morestack` 捕获并抛出错误
9. 恢复抢占

可以看到 `reentersyscall` 多次调用 `save` 保存 `pc` 和 `sp`。`save` 更新 `getg().sched` 中的 `sp` 和 `pc` ，使得调用 `gogo` 的时候可以恢复 `pc` 和 `sp` 。`reentersyscall` 中 `save` 的目的都是为 goroutine 跳回这个 `syscall` 调用者执行 `syscall` 时刻的 `pc` 和 `sp`做准备。

需要继续深入：
1. `StackPreempt` 
2. `syscallstack` 和 `syscallguard` 的具体作用时机

# `runtime·entersyscallblock`

与 `runtime·entersyscall` 区别在于这个函数认为当前执行的 `syscall` 会运行较长时间，因此在函数中主动进行了 M 和 P 的解除绑定，无需等待 `sysmon` 处理。解除 M 和 P 绑定的逻辑由 `entersyscallblock_handoff` 实现。

```go
func entersyscallblock_handoff() {
	if trace.enabled {
		traceGoSysCall()
		traceGoSysBlock(getg().m.p.ptr())
	}
	handoffp(releasep())
}
```

# `runtime·exitsyscall`

主要实现了从 `syscall` 状态中恢复的动作：
1. 尝试调用 `exitsyscallfast` ，如果 M 与 P 没有完全解除绑定，那么该操作会将 M 和 P 重新绑定；否则获取一个空闲的 P 与当前 M 绑定。如果绑定成功，返回 `True`，否则返回 `False` 进行后续步骤处理。
2. 如果 `exitsyscallfast` 返回 `True` ，函数就直接返回；返回 `False`，则进入 *slow path* 将当前 goroutine 放到任务队列中等待调度，具体实现由 `mcall(exitsyscall0)` 实现。

`exitsyscall0` 这个函数比较清晰，只是对其中 `dropg()` 的目的还没想清楚。

`runtime·exitsyscall` 的函数说明中提到的 `// Write barriers are not allowed because our P may have been stolen.` 也没有搞清楚，知道和 GC 有一定关系。




# 引用
1. https://studygolang.com/articles/7005

