+++
title = "Go by Example: Atomic & Mutexes"
date = "2026-02-18"
description = ""
draft = false

[taxonomies]
tags = ["golang", "atomic", "mutex"]
+++

# 参考资料
- [gobyexample: atomic-counters](https://gobyexample.com/atomic-counters)
- [gobyexample: mutexes](https://gobyexample.com/mutexes)
- [gobyexample: stateful-goroutines](https://gobyexample.com/stateful-goroutines)


# Atomic
可以使用`sync/atomic`包来实现允许多个`goroutine`并发访问的原子计数。

> The primary mechanism for managing state in Go is communication over channels.

```go
import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    // 原子计数器，其零值为0
    var counter atomic.Uint64
    // 使用WaitGroup确保所有goroutine执行完毕再统计
    var wg sync.WaitGroup
    // 创建50个goroutine,每个都修改计数
    for range 50 {
        wg.Go(func() {
            for range 1000 {
                counter.Add(1) // 原子自增
            }
        })
    }

    wg.Wait() // 等待所有goroutine完成

    fmt.Println("counter:", counter.Load()) // counter: 50000
}

```


# Mutex
可以使用互斥锁（`mutex`）实现安全的跨`goroutine`访问数据。注意**锁不能被拷贝**，如果需要作为参数，只能通过指针访问。

```go
// 我们想实现并发读写map,可以为其添加一个锁
type Container struct {
    mu       sync.Mutex
    counters map[string]int
}

func (c *Container) inc(name string) {
    c.mu.Lock() // 首先获取锁
    defer c.mu.Unlock() // 使用defer确保函数退出时释放锁
    c.counters[name]++
}
```

## 互斥锁的公平性
> 以下翻译自 `/usr/lib/go/src/internal/sync/mutex.go`

互斥锁（Mutex）有两种工作模式：普通模式（normal）和饥饿模式（starvation）。

在普通模式下，等待者按先进先出（FIFO）的顺序排队，但被唤醒的等待者并不会直接获得互斥锁的所有权，而是需要与新到来的 goroutine 竞争锁的所有权。新到来的 goroutine 具备天然优势 —— 它们已经在 CPU 上运行，且数量可能众多，因此被唤醒的等待者很可能竞争失败。这种情况下，该等待者会被排到等待队列的队首。如果某个等待者尝试获取锁的时间超过 1 毫秒仍未成功，它会将互斥锁切换至饥饿模式。

在饥饿模式下，互斥锁的所有权会由解锁的 goroutine 直接移交至等待队列队首的等待者。新到来的 goroutine 即使看到锁处于未锁定状态，也不会尝试获取锁，更不会执行自旋（spin）操作；相反，它们会直接排到等待队列的队尾。

当某个等待者获得互斥锁所有权后，若满足以下任一条件，会将互斥锁切换回普通模式：(1) 它是等待队列中的最后一个等待者；(2) 它的等待时间不足 1 毫秒。

普通模式的性能表现显著更优，因为即便存在阻塞的等待者，某个 goroutine 也可能连续多次获取到互斥锁。而饥饿模式的核心作用是避免出现尾部延迟（tail latency）的极端异常场景。


# Stateful Goroutines

```go
package main

import (
	"fmt"
	"math/rand/v2"
	"sync"
	"time"
)

type ReadOp struct {
	key  int
	resp chan int // 将通道作为oneshot系统，用来发送&接收返回值
}

type WriteOp struct {
	key  int
	val  int
	resp chan bool // 将通道作为oneshot系统，用来发送&接收返回值
}

func main() {
	reads := make(chan ReadOp, 5)
	writes := make(chan WriteOp, 5)
	quit := make(chan struct{}) // 不发送数据的通道，用通道开关表示是否退出

	var wg sync.WaitGroup

	go func() {
		state := map[int]int{}
		rcounter := 0
		wcounter := 0
		
		// 如果使用下面这行，则永远输出的是0
		// defer 语句在定义时就会立即计算并绑定所有参数的值，而不是在执行时
		// defer fmt.Println("read_counter: ", rcounter, ", write_counter: ", wcounter)
		
		defer func() {
			// 统计读写操作个数
			fmt.Println("read_counter: ", rcounter, ", write_counter: ", wcounter)
			close(quit)
		}()
		
		for {
			select {
			case op := <-reads:
				op.resp <- state[op.key]
				rcounter += 1

			case op := <-writes:
				state[op.key] = op.val
				op.resp <- true
				wcounter += 1

			case <-time.After(1 * time.Second):
				return
			}
		}

	}()

	for range 123 {
		wg.Go(func() {
			readOp := ReadOp{
				key:  rand.IntN(5),
				resp: make(chan int),
			}

			reads <- readOp
			<-readOp.resp
		})
	}

	for range 321 {
		wg.Go(func() {
			writeOp := WriteOp{
				key:  rand.IntN(5),
				val:  rand.IntN(100),
				resp: make(chan bool),
			}

			writes <- writeOp
			<-writeOp.resp
		})
	}

	
	wg.Wait()
	
	<-quit
}

```