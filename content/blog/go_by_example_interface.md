+++
title = "Go by Example: Interfaces"
date = "2026-02-15"
description = ""
draft = false

[taxonomies]
tags = ["golang", "interface"]
+++


# 参考资料
- [gobyexample: interfaces](https://gobyexample.com/interfaces)
- [Go Data Structures: Interfaces](https://research.swtch.com/interfaces)

# Interfaces
接口是方法签名的具名集合，用来实现鸭子类型，接口本身也是一个类型`interface{}`，`any`类型是其别名。
```go
// /usr/lib/go/src/builtin/builtin.go
type any = interface{}
```

若一个结构体实现了该接口的全部方法，则该结构体实现了接口。
若该结构体存在指针实现的接口方法，则作为接口参数传入时必须传入引用，否则无法通过编译。
```go
// 定义接口
type Geometry interface {
    area() float64    // 接口方法
    perim() float64
    scale(factor float64)
}

// 接口作为参数
func measure(g Geometry) {
    fmt.Println(g)
    fmt.Println(g.area())
    fmt.Println(g.perim())
}

type Rect struct {
    width float64
    height float64
}

// 实现接口对应方法
func (self Rect) area() float64 {
    return self.height * self.width
}

// 实现接口对应方法
func (self Rect) perim() float64 {
    return 2 * self.height + 2 * self.width
}

// 指针接收者实现接口方法
func (self *Rect) scale(factor float64) {
    self.height *= factor
    self.width *= factor
}

// 判断接口传入的实际类型
func detectCircle(g Geometry) {
    if c, ok := g.(Rect); ok {
        fmt.Println("Rect with radius", c.radius)
    }
}


func main() {
    rect := Rect{width: 10, height: 20}
    // detectCircle(rect) // 报错！存在接口方法由指针接收者实现
    detectCircle(&rect)   // 正常
}
```

接口类型也可以作为参数。`switch`也可以将类型作为参数。
```go
func test_interface(any interface{}) float64 {
    if v, ok := any.(Geometry); ok {
        return v.area()
    }
    
    switch v := any.(type) {
    case int:
        return 0.0
    case float:
        return 0.0
    }
}
```


# 底层实现
`interface{}`元数据包含`itable`指针和`data`指针，占用16B（x64）。`itable`指针指向一段堆内存，存储实际类型和所有接口方法的地址。`data`指针指向一段堆内存，存储实际类型的元数据。

```go
iface := Geometry(&rect)
fmt.Println("size of interface: ", unsafe.Sizeof(iface))
// size of interface:  16
```

{% mermaid() %}
graph TD
    %% 接口变量层
    subgraph InterfaceValue ["Interface Value (16 bytes)"]
        direction LR
        A["itable: 8 bytes"]
        B["data: 8 bytes"]
    end

    %% itable层
    subgraph Itable ["runtime.itable"]
        direction LR
        A1["typ: *runtime._type"]
        A2["fun[0]: func ptr"]
        A3["fun[1]: func ptr"]
    end

    %% 类型信息层
    subgraph TypeInfo ["runtime._type (T's type info)"]
        direction LR
        TI1["name: T"]
        TI2["size: XX bytes"]
        TI3["kind: ptr/struct"]
    end

    %% 具体值层
    subgraph ConcreteValue ["Concrete Value (*T)"]
        direction LR
        C1["Pointer to T instance"]
    end

    %% 结构体实例层
    subgraph TInstance ["T struct instance"]
        direction LR
        TI["Field1: value1"]
        TII["Field2: value2"]
    end

    %% 连线
    A --> Itable
    B --> ConcreteValue
    A1 --> TypeInfo
    C1 --> TInstance
    A2 --> M1["(*T).Method1()"]
    A3 --> M2["(*T).Method2()"]

    %% 简化样式
    classDef core fill:#f0f8ff,stroke:#20b2aa,stroke-width:1px;
    class InterfaceValue,Itable,ConcreteValue core;

{% end %}

`golang`在编译器会优化内存占用，例如接口无方法时不额外分配堆内存，仅仅保存实际类型；数据类型小于等于指针大小时直接存储其值。

`itable`表是在运行时动态查询和缓存的，通过编译时预先排序，运行时只需要`O(m + n)`的复杂度，其中m是接口的方法个数，n是实际类型的方法个数。


