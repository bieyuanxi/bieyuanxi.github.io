+++
title = "Go by Example: Goroutines & Channels"
date = "2026-02-17"
description = ""
draft = false

[taxonomies]
tags = ["golang", "goroutine", "WaitGroups", "channels", "sync"]
+++

# 参考资料
- [gobyexample: goroutines](https://gobyexample.com/goroutines)
- [gobyexample: channels](https://gobyexample.com/channels)
- [gobyexample: channel-buffering](https://gobyexample.com/channel-buffering)
- [gobyexample: channel-directions](https://gobyexample.com/channel-directions)
- [gobyexample: closing-channels](https://gobyexample.com/closing-channels)
- [gobyexample: channel-synchronization](https://gobyexample.com/channel-synchronization)
  - [gobyexample: range-over-channels](https://gobyexample.com/range-over-channels)
- [gobyexample: waitgroups](https://gobyexample.com/waitgroups)



# Goroutines
`goroutine`是一种轻量级的执行线程，由 Go 运行时调度。要想将一个函数`fn`放到`goroutine`上运行，使用`go fn(param)`语句，之后该`goroutine`会和调用线程并发运行。也可以使用`go`关键字运行匿名函数。

```go
func fn(param string) {
	// ...
}

go fn("goroutine")	// 立刻运行

go func(p1 int) {		// 立刻运行匿名函数
	// ...
}(100)
```

# Channels
通道（Channel）是连接并发执行的 `goroutine` 的管道。可以从一个 `goroutine` 向通道中发送值，并在另一个 `goroutine` 中接收这些值。

可以使用`make(chan val-type)`语句创建一个新的通道，其中`val-type`是通道要发送的值的类型，它是通道类型的一部分。
```go
messages := make(chan string)
```

通道`chan val-type`是双向的，既可以发送也可以接收。

使用`channel <-`语法将一个值发送到通道里。`goroutine`可与通道配合使用。
```go
go func() { messages <- "ping" }()
```

使用`<-channel`语法从通道接收值。默认情况下发送和接收会阻塞，直到发送者和接受者准备就绪。
```go
msg := <- messages
fmt.Println(msg)
```

当通道作为函数参数时，可以指明通道只用来发送/接收值。编译器会区分只读通道 `<-chan int` 和 只写通道 `chan<- int`。
```go
// 只读
func consumer(ch <-chan int) {
	val := <-ch
	fmt.Println("read:", val)
}

// 只写
func producer(ch chan<- int) {
	ch <- 100
}

ch := make(chan int)

go producer(ch)
go consumer(ch)
```


## Buffered Channels
默认情况下，通道是无缓冲的（`unbuffered`），这意味着只有当存在对应的接收操作（`<- chan`）已准备好接收发送的值时，通道才会接受发送操作（`chan <-`）。而有缓冲通道（`buffered channels`）则可以在没有对应接收方的情况下，接收有限数量的值。即使通道已经被关闭，依然可以获取通道缓存的值。

创建一个有缓冲通道很简单。例如下面创建了一个能缓冲2个值的通道。
```go
func main() {

    messages := make(chan string, 2)

    messages <- "buffered"
    messages <- "channel"
    // 注意这里发送了两个值，刚好没有阻塞
    // 如果再多发一个，在当前代码下会发生死锁
    // fatal error: all goroutines are asleep - deadlock!

    fmt.Println(<-messages)
    fmt.Println(<-messages)
}
```


## Channel Synchronization
可以使用通道来同步不同 `goroutine` 之间的执行，例如通过使用阻塞接收器来等待`goroutine`执行完毕。`WaitGroup`可用于等待多个`goroutine`执行完毕。

```go
func worker(done chan bool) {
    fmt.Print("working...")
    time.Sleep(time.Second)
    fmt.Println("done")
    // Send a value to notify that we’re done.
    done <- true
}

func main() {
    // Start a worker goroutine, giving it the channel to notify on.
    done := make(chan bool, 1)
    go worker(done)
    // lock until we receive a notification from the worker on the channel.
    <-done
    // 如果没有<-done，程序会在worker完成之前退出
}
```


## Closing Channels
关闭一个通道意味着该通道上不再发送任何值。这可用于向通道的接收方传达完成信号。

从一个已关闭的通道读取值会立刻返回，第一个返回值返回发送的值或零值，第二个可选参数负责区分读取的值是否有效，`true`表示有效，`false`表示无效。
```go
func main() {
	jobs := make(chan int, 5)
	done := make(chan bool)

	go func() {
		for {
			// 返回值more表示接收到的值是否有效
			j, more := <-jobs
			if more {
				fmt.Println("received job", j)
			} else {
				fmt.Println("received all jobs")
				done <- true
				return
			}
		}
	}()

	go func() {
		for j := range 300 {
			jobs <- j
			fmt.Println("sent job", j)
		}
		close(jobs)
		fmt.Println("sent all jobs")
	}()

	<-done
  // 验证通道状态，应该已经关闭
	_, ok := <-jobs
	fmt.Println("received more jobs:", ok) // received more jobs: false
}
```


## Range over Channels
可以使用`range`从通道迭代值。当不可能再获取值时，循环将自动退出。
```go
func main() {
    queue := make(chan string, 2)
    queue <- "one"
    queue <- "two"
    close(queue)

    for elem := range queue {
        fmt.Println(elem)
    }
}
```


## WaitGroups
为了等待多个`goroutines`完成，可以使用`sync.WaitGroups`。
需要注意的是，如果`WaitGroups`需要显式作为函数参数，应该使用指针形式。

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int) {
    fmt.Printf("Worker %d starting\n", id)

    time.Sleep(time.Second)
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    var wg sync.WaitGroup

    // Launch several goroutines using WaitGroup.Go
    for i := 1; i <= 5; i++ {
        wg.Go(func() {
            worker(i)
        })
    }
    // Block until all the goroutines started by wg are done.
    wg.Wait()

}
```

`WaitGroup`没有直接传播错误的方式，可以使用`errgroup`包。
