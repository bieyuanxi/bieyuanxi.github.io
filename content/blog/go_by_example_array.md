+++
title = "Go by Example: Arrays"
date = "2026-02-14"
description = ""
draft = true

[taxonomies]
tags = ["golang", "array"]
+++


# 参考资料
- [gobyexample: arrays](https://gobyexample.com/arrays)



# 使用方法
在 Go 语言中，数组是**长度固定的**、由编号序列构成的特定类型元素集合，数组的大小完全取决于其数组类型（`[len]type`），在编译时即已知，且无法动态改变大小。编译器会根据逃逸分析结果、预定义的数组大小等因素决定是在堆上还是栈上分配空间。在常规的 Go 代码编写中，切片的使用要普遍得多；数组仅在一些特殊场景下才会发挥作用。

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
fmt.Println(b)
```

```sh
[100 0 0 400 500]
```


创建并初始化多维数组：
```go
twoD := [2][3]int{
    {1, 2, 3},
    {1, 2, 3},
}
```