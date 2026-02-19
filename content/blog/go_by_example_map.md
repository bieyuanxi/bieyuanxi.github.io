+++
title = "Go by Example: Maps"
date = "2026-02-12"
description = ""
draft = false

[taxonomies]
tags = ["golang", "hashmap", "map"]
+++

# 参考资料
- [gobyexample: maps](https://gobyexample.com/maps)

Golang中的`map[key-type]val-type`类型语义上是一个指针，大小等于操作系统指针的大小，因此其[`零值`](https://go.dev/ref/spec#The_zero_value)为`nil`。因此使用`new`创建`map`是没有意义的。可以使用如下代码验证：
```go
import (
    "fmt"
    "unsafe"
)

mapSize := unsafe.Sizeof(m)
fmt.Printf("map[string]int大小：%d 字节\n", mapSize)
```

# 使用方法

要创建一个空`map`，只需要使用内置的`make`函数：`make(map[key-type]val-type)`。也可以用语法2声明并初始化一个新的`map`。
```go
m := make(map[string]int)
// 第二个参数capacity是容量提示（不是限制），用于预分配内存，提升后续添加元素的性能
// 省略第二个参数时，Go 会使用默认的初始容量创建 map
m := make(map[K]V, capacity)

// syntax 2
n := map[string]int{}
n := map[string]int{"foo": 1, "bar": 2}    // 初始化有2个键-值对
``` 

> `make`是 `Go` 语言的内置函数，专门用于创建切片（`slice`）、映射（`map`）和通道（`channel`）这三种引用类型。

使用经典的`name[key] = val`语法设置键-值对。对于不存在的键-值对自动创建，对于存在的则覆盖。
```go
m["k1"] = 7
m["k2"] = 13
```

可以使用`fmt.Println`打印所有键-值对。注意`map`以`map[k:v k:v]`的形式打印。
```go
fmt.Println("map:", m)
```

```sh
map: map[k1:7 k2:13]
```

使用`name[key]`获取键对应的值。如果键对应的值不存在，则返回值类型的[`零值`](https://go.dev/ref/spec#The_zero_value)。存在一个可选的第二个返回值（`bool`类型），用来在获取值的同时指明该值是否存在。如果只关注是否存在对应的键-值对，可以用下划线`_`忽略返回值。

```go
v1 := m["k1"]

_, exist := m["k2"]
fmt.Println("is key-value exists:", exist)

if _, exist := m["k2"]; exist {
// 存在键-值对
}
```

内置的`len`函数返回键-值对的数量。
```go
fmt.Println("len:", len(m))
```

内置的`delete`函数移除指定的键-值对。
```go
delete(m, "k2")
```

内置的`clear`函数可以删除所有键-值对。
```go
clear(m)
```

`maps`包实现了一堆有用的辅助函数，使用时需要提前导入。
```go
import (
  "maps"
)
n1 := map[string]int{"foo": 1, "bar": 2}
n2 := map[string]int{"foo": 1, "bar": 2}
if maps.Equal(n1, n2) {
    fmt.Println("n1 == n2")
}
// TODO： 其他辅助函数
```


# Golang map底层实现
TODO