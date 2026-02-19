+++
title = "Go by Example: Defer"
date = "2026-02-19"
description = ""
draft = true

[taxonomies]
tags = ["golang", "defer"]
+++

# 参考资料
- [gobyexample: defer](https://gobyexample.com/defer)
- [一文搞懂Go语言中defer的使用](https://juejin.cn/post/7145728803896033311)
- [深入浅出 Go 语言的 defer 机制](https://zhuanlan.zhihu.com/p/689615742)


# Defer
`defer`允许推迟函数的执行直到包含它的函数即将返回，通常用于清理资源和错误处理。
```go
func readFile(filename string) error {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    // 确保文件在函数返回时关闭
    defer f.Close()

    // ...

    return nil
}
```

在一个函数内允许使用多个`defer`，按照`FILO`顺序执行。
```go
func defer_fn_call() {
    defer fn1() // 执行顺序3
    defer fn2() // 执行顺序2
    defer fn3() // 执行顺序1
}
```

`defer`语句会计算并绑定所有参数的值。具体来说，在把 `defer` 压入“栈”时，会同时压入函数地址和函数形参，也就是会在这个时候就把参数先算好。
```go
func defer_print_count() {
    cnt := 1
    // 将闭包和参数c1压入延迟栈中，cnt是闭包通过环境变量捕获的
    defer func(c1 int) {
        fmt.Println("cnt:", cnt, "c1:", c1) // cnt: 2 c1: 100
    }(cnt * 100) // 计算得到参数为100，压栈
    
    cnt += 1
}

func defer_print_count() {
    cnt := 1
    defer fmt.Println("cnt:", cnt) // cnt: 1
    
    cnt += 1
}
```

对于`defer`语句的函数参数包含子函数的情况，子函数会先执行。
```go
// 执行顺序：fn2, fn4, fn3, fn1
defer fn1(fn2())
defer fn3(fn4())
```

# defer 与 return
`defer`甚至可以通过闭包对返回值做出修改。`defer`在`return`指令执行之前被执行。
```go
func getI() (i int) {
    i = 1
    defer func() {
        i *= 10
    }()
    return 20
}

func main() {
    fmt.Println(getI()) // 200
}

```

# defer 与 错误处理
`defer`可以配合`recover()`捕获`panic`，从而处理错误。下面的代码可以看出，即使`panic`，defer里的代码依然被执行。
```go
func panicAndRecover() {
    defer func() {
        if e := recover(); e != nil {
            // Huh~ Just recovered! It says: panic!
            fmt.Println("Huh~ Just recovered! It says:", e)
        }
    }()
    
    panic("panic!")
}

```
如果在`defer`语句中也有`panic`，则会继续向上传播，但是`panic`只会保存最新抛出的错误。


# defer实现机制
`defer`存在堆上分配、栈上分配、开放编码三种方式。
TODO



