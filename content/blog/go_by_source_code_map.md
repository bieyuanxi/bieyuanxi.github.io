+++
title = "Go by Source Code: Map"
date = "2026-03-06"
description = ""
draft = false

[taxonomies]
tags = ["golang", "source code", "hashmap", "map"]
+++

# 参考资料
- [abseil.io: swisstables](https://abseil.io/about/design/swisstables)
- [golang go1.24.0](https://github.com/golang/go/blob/go1.24.0/src/runtime/map_noswiss.go)
- [小林coding | Map面试题](https://xiaolincoding.com/interview/golang.html#_3-map%E9%9D%A2%E8%AF%95%E9%A2%98)
- [The new builtin map implementation](https://go.dev/doc/go1.24#runtime)

# Golang map底层实现
> golang 在`1.24`版本基于[Swiss Tables](https://abseil.io/about/design/swisstables)表重新实现了`map`，具体参阅：[The new builtin map implementation](https://go.dev/doc/go1.24#runtime)。在`1.24`版本，旧的实现文件被重新命名为`map_noswiss.go`，新实现命名为`map_swiss.go`，由`goexperiment.swissmap`控制开启。


## go1.24.0
`golang`的`map[key]val`类型实际上是一个指针，指向的就是`hmap`结构体，其中`buckets`指针对应的数组保存的是桶，每个桶的内存布局以紧密排布方式存储8个键-值对和8个`tophash`，先存8个`tophash`（`hash`值前8位），再存8个`key`，接着是8个`value`，之后是对齐，最后是溢出桶的指针。
```go
// src/runtime/map_noswiss.go
// A header for a Go map.
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/reflectdata/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
	clearSeq   uint64

	extra *mapextra // optional fields
}

// A bucket for a Go map.
type bmap struct {
	// tophash generally contains the top byte of the hash value
	// for each key in this bucket. If tophash[0] < minTopHash,
	// tophash[0] is a bucket evacuation state instead.
	tophash [abi.OldMapBucketCount]uint8
	// Followed by bucketCnt keys and then bucketCnt elems.
	// NOTE: packing all the keys together and then all the elems together makes the
	// code a bit more complicated than alternating key/elem/key/elem/... but it allows
	// us to eliminate padding which would be needed for, e.g., map[int64]int8.
	// Followed by an overflow pointer.
}
```
### 查询
桶的个数始终是2的幂次个，查询时首先对`hash`值低若干位取掩码获取桶索引。举例：桶个数为16 = 2^4，则对hash值低4位取掩码获得桶索引。之后遍历桶的8个`tophash`寻找对应的键-值对，注意到`tophash`只有8位，所以存在`tophash`相同但是`hash`不同的情况，所以还需要比较`key`是否相同（即使`hash`相同也可能会碰撞），如果存在溢出桶，还会继续到溢出桶继续这个流程。


### 扩容
扩容不是一次性完成的，会在每次`map`进行操作时搬移一两个桶（TODO）。

#### 触发条件
有两种情况会触发扩容（前提是当前不处于扩容状态）：
1. 键-值对总数超过单桶容量(8)且装载因子超过`6.5`时；
```go
// 条件1

// src/internal/abi/map.go
// 11:	MapBucketCountBits = 3 // log2 of number of elements in a bucket.
// 12:	MapBucketCount     = 1 << MapBucketCountBits

// 避免浮点运算
const loadFactorNum invalid type = loadFactorDen * abi.MapBucketCount * 13 / 16 // Numerator（分子） 
const loadFactorDen untyped int = 2	// Denominator（分母）

// overLoadFactor reports whether count items placed in 1<<B buckets is over loadFactor.
func overLoadFactor(count int, B uint8) bool {
	// loadFactorNum*(bucketShift(B)/loadFactorDen)的表示避免了浮点运算
	return count > abi.MapBucketCount && uintptr(count) > loadFactorNum*(bucketShift(B)/loadFactorDen)
}
```

2. 溢出桶太多时，如果桶总数大于`2^B`个，按照`2^15`个考虑，否则按照`2^B`个计算溢出桶数量，记为`N`，当溢出桶数量大于等于`N`时触发扩容；
```go
// 条件2

// tooManyOverflowBuckets reports whether noverflow buckets is too many for a map with 1<<B buckets.
// Note that most of these overflow buckets must be in sparse use;
// if use was dense, then we'd have already triggered regular map growth.
func tooManyOverflowBuckets(noverflow uint16, B uint8) bool {
	// If the threshold is too low, we do extraneous work.
	// If the threshold is too high, maps that grow and shrink can hold on to lots of unused memory.
	// "too many" means (approximately) as many overflow buckets as regular buckets.
	// See incrnoverflow for more details.
	if B > 15 {
		B = 15
	}
	// The compiler doesn't see here that B < 16; mask B to generate shorter shift code.
	return noverflow >= uint16(1)<<(B&15)
}
```


## After go1.24.0

TODO