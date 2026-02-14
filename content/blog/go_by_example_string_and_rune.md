+++
title = "Go by Example: Strings and Runes"
date = "2026-02-15"
description = ""
draft = false


[taxonomies]
tags = ["golang", "string", "rune", "char"]
+++

# 参考资料
- [gobyexample: Strings and Runes](https://gobyexample.com/strings-and-runes)


Go 中的字符串`string`是一段**只读**的字节切片，由指针+长度组成，占用16B(x64)。语言本身与标准库会对字符串做特殊处理 —— 将其视为以 `UTF-8` 编码的文本容器。
在其他语言中，字符串由 “字符（character）” 构成；而在 Go 里，对应 “字符” 的概念叫作 `rune`：它是一个整数(`uint32`的别名)，代表一个 `Unicode` 码点（Unicode code point）。
`rune`的定义和行为和`Rust`的`char`非常相似。

```go
const hello = "你好！Terra！"	// const hello untyped string
world := "世界"	// var world string
r := '🌏'
fmt.Printf("%c\n", r)	// 🌏
```

golang区分`untyped string`（无类型字符串常量）和`string`（字符串类型）。
`untyped string`是编译期常量，存储在程序的只读数据区，不可修改，没有内存地址。
`string`有明确的内存地址（变量存在栈 / 堆，值指向只读数据区）。
```go
var s string = "hello" // "hello" 是 untyped string，赋值后 s 成为 typed string
```




`string`底层用`[]byte`存储，使用内置函数`len`会获取实际占用的裸字节大小。对字符串进行索引操作，会返回每个索引位置上的原始字节值。这个循环会生成字符串中所有构成 `Unicode` 码点的字节对应的十六进制值。
```go
fmt.Println("Len:", len(hello))	// Len: 17

for i := 0; i < len(s); i++ {
    fmt.Printf("%x ", s[i]) // e4 bd a0 e5 a5 bd ef bc 81 54 65 72 72 61 ef bc 81
}
fmt.Println()
```

要统计`string`中字符`rune`的个数，可以使用`utf8`包的`RuneCountInString`函数。需要注意的是运行所需的时间取决于`string`的大小，因为需要逐个分割`UTF-8`字符并转为`rune`类型。
```go
import (
  "unicode/utf8"
)

fmt.Println("Rune count: ", utf8.RuneCountInString(hello))	// Rune count: 9
```

在循环中可以使用`range`关键字自动解码为`rune`。
A range loop handles strings specially and decodes each rune along with its offset in the string.
```go
for idx, r := range hello {
	// fmt.Println("", r)	// 直接打印rune类型只会获得一个数字
	// fmt.Printf("%c starts at %d\n", r, idx)	// 使用%c
	fmt.Printf("%#U starts at %d\n", r, idx)
}
```
```sh
U+4F60 '你' starts at 0
U+597D '好' starts at 3
U+FF01 '！' starts at 6
U+0054 'T' starts at 9
U+0065 'e' starts at 10
U+0072 'r' starts at 11
U+0072 'r' starts at 12
U+0061 'a' starts at 13
U+FF01 '！' starts at 14
```


可以使用`+`生成一个新的`string`（频繁操作会导致大量内存分配和拷贝，性能极差）。
```go
s1 := "123"
s2 := "abc"
s3 := s1 + s2
fmt.Println(s3) // 123abc
```


`strings`包包含了一些方便的函数。
```go
import (
	"strings"
)

b := strings.Builder{}
b.Grow(128) // 预分配 128 bytes
b.WriteString("hello")
b.WriteString("world")
b.WriteRune('!')
```