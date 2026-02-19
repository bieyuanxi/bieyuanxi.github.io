+++
title = "Go by Example: Sorting"
date = "2026-02-17"
description = ""
draft = false

[taxonomies]
tags = ["golang", "sorting"]
+++


> 在`1.21`版本及之后，推荐使用`slices`包里的排序相关算法，而不是使用`sort`包的，尤其是一些非泛型的排序算法。
```go
// Note: as of Go 1.22, this function simply calls [slices.Sort].
func Float64s(x []float64) { slices.Sort(x) }
```

# 参考资料
- [gobyexample: sorting](https://gobyexample.com/sorting)
- [gobyexample: sorting-by-functions](https://gobyexample.com/sorting-by-functions)


# Sorting
`golang`的`slices`包实现了内置类型和用户定义类型的排序算法。排序函数`Sort`接受泛型数组作为参数，以自然顺序和升序排序，要求类型实现`cmp.Ordered`接口，适用于内置的有序类型。

`Ordered` 接口约束的类型必须是底层类型为整数 / 浮点数 / 字符串的类型，称之为有序类型，这些类型的共同特征是支持`<` `<=` `>` `>=`等大小比较操作。
```go
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
        ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr |
        ~float32 | ~float64 |
        ~string
}
```
> 近似类型（`~T`）：匹配 “底层类型为 `T`” 的所有类型（包括自定义类型别名），比如`type MyInt int` 的底层类型是 `int`，会匹配`~int`


```go
package main

import (
    "fmt"
    "slices"
)

func main() {
    strs := []string{"c", "a", "b"}
    slices.Sort(strs)
    fmt.Println("Strings:", strs) // Strings: [a b c]

    ints := []int{7, 2, 4}
    slices.Sort(ints)
    fmt.Println("Ints:   ", ints) // Ints:    [2 4 7]

    s := slices.IsSorted(ints)
    fmt.Println("Sorted: ", s)    // Sorted:  true
}
```


# Sorting By Functions
前面的`slices.Sort`函数只适用于内置的有序类型，而且只能以自然顺序和升序方式排列。`slices.SortFunc`函数可以对任何切片数组排序，且可以自定义排序方式，例如以长度作为排序标准。


首先需要实现一个比较函数。这里使用`cmp.Compare`函数，该函数需要传入两个可比较的值，即`Ordered`接口约束的基础类型。
```go
import (
  "cmp"
)

lenCmp := func(a, b string) int {
      //    -1 if x is less than y,
    //     0 if x equals y,
    //    +1 if x is greater than y.
    return cmp.Compare(len(a), len(b))
}
```

之后可以通过调用`slices.SortFunc`函数并传入自定义比较函数来排序。
```go
slices.SortFunc(fruits, lenCmp)
fmt.Println(fruits)
```

对于非内置类型也可以使用相同的方法。不过，如果非内置结构体很大，最好不要直接以值传递的方式排序，而是对原切片创建指针数组或索引数组，先对数组排序，之后拷贝排序后的结果。
```go
// 函数签名要求排序函数和传入的切片具有相同的类型
func SortFunc[S ~[]E, E any](x S, cmp func(a, b E) int) {
    // ...
}
```

```rust
// rust的函数签名，要求排序函数的参数是元素的引用
pub fn sort_by<F>(&mut self, compare: F)
where
    F: FnMut(&T, &T) -> Ordering,
```
总而言之，对大型结构体进行排序，需要考虑值传递的开销。`golang`要求比较函数的参数和切片元素具有相同的类型是有一定的道理的。


`slices`包里和排序有关的还有一些算法。
```go
func Sort[S ~[]E, E cmp.Ordered](x S) {
    // 
}

func SortFunc[S ~[]E, E any](x S, cmp func(a, b E) int) {
    //
}

func SortStableFunc[S ~[]E, E any](x S, cmp func(a, b E) int) {
    // ...
}

func IsSorted[S ~[]E, E cmp.Ordered](x S) bool {
    //
}

func IsSortedFunc[S ~[]E, E any](x S, cmp func(a, b E) int) bool {
    //
}

// It panics if x is empty.
func Min[S ~[]E, E cmp.Ordered](x S) E {
    // ...
}

func MinFunc[S ~[]E, E any](x S, cmp func(a, b E) int) E {
    //
}

func Max[S ~[]E, E cmp.Ordered](x S) E {
    //
}

func MaxFunc[S ~[]E, E any](x S, cmp func(a, b E) int) E {
    //
}

func BinarySearch[S ~[]E, E cmp.Ordered](x S, target E) (int, bool) {
    //
}

func BinarySearchFunc[S ~[]E, E, T any](x S, target T, cmp func(E, T) int) (int, bool) {
    // 
}
```