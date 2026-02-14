+++
title = "Go by Example: Arrays & Slices"
date = "2026-02-14"
description = "Slice = ptr + len + cap"
draft = false

[taxonomies]
tags = ["golang", "array", "slice"]
+++


# 参考资料
- [gobyexample: arrays](https://gobyexample.com/arrays)
- [gobyexample: slices](https://gobyexample.com/slices)
- [Go Slices: usage and internals](https://go.dev/blog/slices-intro)


# Array
在 Go 语言中，数组是**长度固定的**、由编号序列构成的特定类型元素序列，数组的大小完全取决于其数组类型（`[len]type`），在编译时即已知，且无法动态改变大小。
编译器会根据逃逸分析结果、预定义的数组大小等因素决定是在堆上还是栈上分配空间。
在常规的 Go 代码编写中，切片的使用要普遍得多，数组仅在一些特殊场景下才会发挥作用。
数组被当作函数参数传递时，会拷贝整个数组的所有元素，可以使用切片或指针避免拷贝整个数组。
> Go’s arrays are values.

可以使用如下代码验证数组大小：
```go
// ...：编译器负责推导数组大小，也可以手动指定大小
// 编译器推导的array类型为 [6]int，大小为6*8=48 B
array := [...]int{
	1, 2, 3, 4, 5, 6,
}

fmt.Println("size of [6]int: ", unsafe.Sizeof(array))
fmt.Println("size of int: ", unsafe.Sizeof(int(5)))
fmt.Println("size of 5: ", unsafe.Sizeof(5))
```

```sh
// x64
size of [6]int:  48
size of int:     8
size of 5:       8
```

可以通过索引指定对应下标的值。
```go
// 将index=3对应的值设置为400,后面的从4开始设置，中间跳过的均设置为零值
b := [...]int{100, 3: 400, 500}
fmt.Println(b)	// [100 0 0 400 500]
```


创建并初始化多维数组：
```go
twoD := [2][3]int{
    {1, 2, 3},
    {1, 2, 3},
}
```


# Slice
切片是 Go 语言中一种重要的数据类型，相比数组，它为序列型数据提供了更为强大的操作接口。
和数组不同，切片的类型只和它包含的元素的类型有关(`[]type`)，一个未初始化的切片等于`nil`，长度为0。
切片类型的大小由指针、长度和容量决定，在64位环境下占用24B。
```go
var s []string
fmt.Println("uninit:", s, s == nil, len(s) == 0)	// uninit: [] true true
```



To create a slice with non-zero length, use the builtin make. Here we make a slice of strings of length 3 (initially zero-valued). By default a new slice’s capacity is equal to its length; if we know the slice is going to grow ahead of time, it’s possible to pass a capacity explicitly as an additional parameter to make.
若要创建一个长度非零的切片，可使用内置函数 `make`。这里我们创建了一个长度为 3 的字符串类型切片（初始值均为零值）。默认情况下，新创建切片的容量等于其长度；如果提前知道该切片后续会扩容，可在调用 `make` 时额外传入一个参数，显式指定切片的容量。
```go
s1 := make([]string, 3)
s2 := make([]string, 3, 10)	// 显式指定切片的容量为10
fmt.Println("emp:", s1, "len:", len(s1), "cap:", cap(s1))	// emp: [  ] len: 3 cap: 3
fmt.Println("emp:", s2, "len:", len(s2), "cap:", cap(s2))	// emp: [  ] len: 3 cap: 10
```

设置切片的值和获取值的方法和数组一致。设置或获取超过切片长度（`len`）的值会`panic`。
```go
s[0] = "a"
s[1] = "b"
s[2] = "c"
fmt.Println("set:", s)
fmt.Println("get:", s[2])

s2[7] = "will panic"	// panic: runtime error: index out of range [7] with length 3

```

使用内置函数`len`获取切片长度。
```go
fmt.Println("len:", len(s))
```


除了这些基础操作外，切片还支持若干更强大的操作，这使得它相比数组功能更丰富。其中一个是内置函数 `append`，该函数会返回一个包含一个或多个新值的切片。需要注意的是，必须接收 `append` 的返回值，因为调用后可能会发生扩容，从而得到一个全新的切片实例。
```go
s := []string{"a", "b", "c"}
fmt.Println("s: ", s, "len:", len(s), "cap:", cap(s))	// s:  [a b c] len: 3 cap: 3
fmt.Printf("addr: %p\n", s)		// addr: 0xc0000a6180
s = append(s, "d")			// 发生扩容！
s = append(s, "e", "f")	// 可以一次添加多个元素
fmt.Println("after append:", s)	// after append: [a b c d e f]
fmt.Printf("addr: %p\n", s)		// addr: 0xc0000901e0
```

> 使用 %p 打印 s 时，显示的是底层数组的首地址；打印 &s 时，显示的是切片结构体本身的地址，这是两个完全不同的地址。


可使用内置函数`copy`拷贝切片。golang允许拷贝涉及的内存区域有重叠的情况，且会正确处理。
```go
src := []string{"a", "b", "c"}
dst := make([]string, len(src))
copy(dst, src)  // 内置函数，返回值是实际拷贝的元素个数
```

切片支持一种名为「切片」的操作，其语法格式为 `slice[low:high]`。例如，通过 `s[2:5]` 可截取得到一个新切片，包含原切片中索引为 2、3、4 的元素（即 s[2]、s[3]、s[4]）。其底层实现只是拷贝指针、长度、容量并修改而已（浅拷贝），并不涉及底层数据的拷贝，这意味着会有多个切片共享一个相同的底层数组。
获取或使用超出底层数组有效范围的切片会`panic`。
存在一种情况：对于一个很大的切片，只引用其中很小的一部分作为切片，这会导致GC无法回收，解决方法是将需要的部分单独复制（使用`copy`或者`append`）。

```go
s := []string{"a", "b", "c", "d", "e", "f"}
s1 := s[2:5]
s2 := s[2:]
s3 := s[0:2]
fmt.Println("s1:", s1, "len:", len(s1), "cap:", cap(s1))	// s1: [c d e] len: 3 cap: 4
fmt.Println("s2:", s2, "len:", len(s2), "cap:", cap(s2))	// s2: [c d e f] len: 4 cap: 4
fmt.Println("s3:", s3, "len:", len(s3), "cap:", cap(s3))	// s3: [a b] len: 2 cap: 6
s4 := s[8:]	// panic: runtime error: slice bounds out of range [8:6]
```


`slices`包包含了一些有用的函数用于操作切片。
```go
import (
    "slices"
)
t2 := []string{"g", "h", "i"}
if slices.Equal(t1, t2) {
	fmt.Println("t1 == t2")
}
```


切片可被组合成多维数据结构。与多维数组不同的是，内层切片的长度可以是可变的。
```go
twoD := make([][]int, 3)
for i := range 3 {
    innerLen := i + 1
    twoD[i] = make([]int, innerLen)
    for j := range innerLen {
        twoD[i][j] = i + j
    }
}
```
