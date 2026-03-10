+++
title = "Kotlin Types: Numbers"
date = "2026-03-10"
description = "旧日的诅咒挥之不去..."
draft = false

[taxonomies]
tags = ["kotlin", "numbers"]
+++



# 参考资料
- [kotlinlang: numbers](https://kotlinlang.org/docs/numbers.html)


# 整型类型

| Type  | Size (bits) | Min value                          | Max value                          |
|-------|-------------|------------------------------------|------------------------------------|
| Byte  | 8           | -128                               | 127                                |
| Short | 16          | -32768                             | 32767                              |
| Int   | 32          | -2,147,483,648 (-2³¹)              | 2,147,483,647 (2³¹ - 1)            |
| Long  | 64          | -9,223,372,036,854,775,808 (-2⁶³) | 9,223,372,036,854,775,807 (2⁶³ - 1) |


```kotlin
val one = 1 // Int
val threeBillion = 3000000000 // Long
val oneLong = 1L // Long
val oneByte: Byte = 1
```

# 浮点类型

| Type   | Size (bits) | Significant bits | Exponent bits | Decimal digits |
|--------|-------------|------------------|---------------|----------------|
| Float  | 32          | 24               | 8             | 6-7            |
| Double | 64          | 53               | 11            | 15-16          |


```kotlin
val pi = 3.14          // Double
val e = 2.7182818284          // Double
val eFloat = 2.7182818284f    // Float, actual value is 2.7182817

// error: Initializer type mismatch: expected 'Double', actual 'Int'.
val one: Double = 1    // Int is inferred 
```

# Boxing and caching numbers on the Java Virtual Machine

`kotlin` 在`JVM`上遵循对数字类型的装箱（`Boxing`）和缓存（`Caching`）机制。`JVM` 原生支持的基础类型（如 `Kotlin` 的 `Int`、`Double`，对应 `Java` 的 `int`、`double`），直接存储数值，不占用对象内存。
当需要把基本类型变成「可空」（如 `Int?`）或用于泛型时，`JVM` 会把基本类型包装成对应的 `Java` 类对象（如 `Int` → `Integer`、`Double` → `Double`），这个过程叫「装箱」。
简单说：`Int` 是数值本身，`Int?` 是装着这个数值的「对象盒子」。

## JVM 的数字缓存优化
为了节省内存，`JVM` 对 -128 ~ 127 范围内的整数（刚好是 `byte` 类型的取值范围）做了特殊处理：
- 这个范围内的数字被装箱成 `Integer` 对象时，不会新建对象，而是复用「缓存池」里的同一个对象；
- 超出这个范围的数字，每次装箱都会新建一个全新的 `Integer` 对象。


观察以下两个例子的结果：
```kotlin
val a: Int = 100          // 基本类型，直接存数值 100
val boxedA: Int? = a      // 装箱，复用缓存池里的 Integer(100) 对象
val anotherBoxedA: Int? = a // 装箱，还是复用同一个 Integer(100) 对象

println(boxedA === anotherBoxedA) // true
```

```kotlin
val b: Int = 10000        // 基本类型，存数值 10000
val boxedB: Int? = b      // 装箱，新建一个 Integer(10000) 对象
val anotherBoxedB: Int? = b // 装箱，又新建一个全新的 Integer(10000) 对象

println(boxedB === anotherBoxedB) // false（不是同一个对象）
println(boxedB == anotherBoxedB)  // true（数值相等）

```
> `===` 是 `Kotlin` 的「引用相等」：判断两个变量是否指向同一个对象，所以，千万不要用 `===` 比较可装箱的数字（如 `Int?`、`Short?` 等）

> `==` 是「结构相等」：判断两个对象的数值是否相同


# 显式类型转换
数字类型(Number)支持显式转换。例如：
- toByte(): Byte (deprecated for Float and Double)
- toShort(): Short
- toInt(): Int
- toLong(): Long
- toFloat(): Float
- toDouble(): Double

```kotlin
val byte: Byte = 1
// OK, literals are checked statically

val intAssignedByte: Int = byte 
// Initializer type mismatch

val intConvertedByte: Int = byte.toInt()

println(intConvertedByte)
```


# 位操作
- shl(bits) – signed shift left
- shr(bits) – signed shift right
- ushr(bits) – unsigned shift right
- and(bits) – bitwise AND
- or(bits) – bitwise OR
- xor(bits) – bitwise XOR
- inv() – bitwise inversion

```kotlin
val x = 1
val xShiftedLeft = (x shl 2)
println(xShiftedLeft)  // 4

val xAnd = x and 0x000FF000
println(xAnd)          // 0
```


