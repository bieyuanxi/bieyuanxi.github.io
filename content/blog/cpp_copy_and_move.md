+++
title = "C++：拷贝与移动"
date = "2025-12-01"
description = "拷贝是拷贝，移动是移动(晕😵"
draft = false

[taxonomies]
tags = ["C++", "拷贝构造函数", "移动构造函数", "拷贝赋值运算符", "移动赋值运算符"]
+++

# 参考资料
[Cpp Reference](https://cppreference.com/w/cpp/language/copy_constructor.html)

# 一、前置知识：左值vs右值
理解移动语义的前提是区分左值（lvalue） 和右值（rvalue）：

> 左值：有名字、能取地址的对象（比如变量、函数返回的引用），可以出现在赋值号左边。

示例：int a = 10; 中，a 是左值。
> 右值：没有名字、不能取地址的临时对象（比如字面量、函数返回的临时值），只能出现在赋值号右边。

示例：10、a + b、string("hello") 都是右值。

C++11 引入右值引用（&&），专门绑定右值，这是移动语义的基础。

# 二、拷贝构造函数
**拷贝构造函数**（Copy Constructor）用一个已存在的对象**深拷贝**创建新对象（逐字节复制 + 资源重新分配），保证新对象和原对象独立，修改一个不会影响另一个。

额...非常抽象的描述，先来看看拷贝构造函数能为我们带来什么便利吧。

```cpp
// 触发拷贝构造函数：
// 1. 用一个对象初始化新对象
MyClass c1;

MyClass c2 = c1;    // 写法1
MyClass c3(c1);     // 写法2
// c1、c2、c3三个对象相互独立

// 2. 作为函数参数按值传递
```

嗯...似乎确实很方便。哎等等，我不定义拷贝构造函数不也一样能用吗？

这是因为编译器有默认的拷贝构造函数实现，如果没有显式定义，编译器会“贴心的”替我们实现一个，即简单地拷贝原对象的值（浅拷贝）。这带来了一个问题：如果我们的对象里有一个指针，它指向了一块堆内存，默认的拷贝构造函数只会把指针拷贝到新对象中，这会出现两个不同的对象指向了同一块内存的情况，如果两个对象在析构函数内释放指向的内存，就会引发`double free`，嘶...好心办坏事了属于是。

所以，在需要管理堆内存等外部资源的情况下，我们必须自己实现拷贝构造函数（或者禁止编译器自动生成）。常见的实现逻辑是深拷贝原对象。

```cpp
class MyClass {
public:
    // 拷贝构造函数：
    // 参数必须是 const 左值引用（const T&），
    // 保证不会修改原对象，并且允许不可变对象作为参数传入
    MyClass(const MyClass& other) {
        // 深拷贝逻辑：复制数据 + 重新分配资源（比如堆内存、文件句柄）
    }
};
```

默认拷贝构造函数的行为是逐成员浅拷贝。内置类型（`int`、`bool`）值拷贝完全安全；对指针 / 引用成员拷贝指针 / 引用本身（而非指向的资源）；对类成员则调用该成员的拷贝构造函数（比如 `std::string` 是深拷贝，安全；自定义资源类如果没写深拷贝，危险）。

# 三、移动构造函数
**移动构造函数**（Move Constructor）用一个右值对象**浅拷贝**创建新对象（“偷” 原对象的资源，而非重新分配），原对象会被置为 “空状态”（资源指针置空），避免不必要的拷贝，提升性能。

```cpp
// 触发移动构造函数：
// 用右值初始化新对象（比如临时对象、std::move 转换的左值）
MyClass c1;
MyClass c2 = std::move(s1);    // 写法1
MyClass c3(std::move(s2));     // 写法2
// c1的资源已被转移给c2
```

> `std::move()` 是一个编译期的类型转换工具，不执行任何运行时的内存移动或数据拷贝操作，核心作用是将左值强制转换为右值引用，从而触发移动语义。

在容器操作中，例如向`vector`放入`string`，使用移动语义可以避免深拷贝对象。

好吧，我承认移动语义确实很实用，那么该如何写移动构造函数呢？

```cpp
class MyClass {
public:
    // 手动编写移动构造函数时，编译器不会自动实现拷贝构造函数

    // 移动构造函数：参数是非 const 右值引用（T&&）
    MyClass(MyClass &&other) noexcept {
        // 移动逻辑：接管原对象资源 + 原对象置空
        println("MyClass::Move function called.");
    }
};
```

> 注：`noexcept` 是 C++11 引入的异常说明符，核心作用是:告诉编译器 / 调用者，该函数不会抛出任何异常，编译器可基于这个承诺做性能优化。如果标记了 `noexcept` 的函数实际抛出异常，程序会直接调用 `std::terminate` 终止。

需要注意的是，在手动编写移动构造函数时，编译器不会自动实现拷贝构造函数。这是因为手动编写移动构造函数意味着我们需要实现资源转移，而资源转移通常都是深拷贝的，编译器默认实现的拷贝构造函数并不合适，因此没有自动生成。如果直接使用拷贝构造函数的语法，则会遇到下面的问题：
- Copy assignment operator is implicitly deleted because 'MyClass' has a user-declared move constructor.
- Copy constructor is implicitly deleted because 'MyClass' has a user-declared move constructor.



# 四、拷贝赋值运算符 vs 移动赋值运算符
构造函数是 “创建新对象”，赋值运算符是 “给已存在的对象赋值”。
- 拷贝赋值运算符（Copy Assignment Operator）
- 移动赋值运算符（Move Assignment Operator）

```cpp
class MyString {
private:
    char* data;
    int len;
public:
    // 普通构造函数
    MyString(const char* str) {
        len = strlen(str);
        data = new char[len + 1]; // 分配堆内存
        strcpy(data, str);
    }

    // 析构函数：释放资源
    ~MyString() {
        delete[] data;
    }

    // 拷贝赋值运算符：返回值为引用（支持链式赋值）
    MyString& operator=(const MyString& other) {
        // 1. 防止自赋值（s = s）
        if (this == &other) return *this;

        // 2. 释放当前对象的资源
        delete[] data;

        // 3. 深拷贝原对象的资源
        len = other.len;
        data = new char[len + 1];
        strcpy(data, other.data);

        return *this;
    }

    // 移动赋值运算符：返回值为引用，参数为右值引用
    MyString& operator=(MyString&& other) noexcept {
        // 1. 防止自赋值
        if (this == &other) return *this;
    
        // 2. 释放当前对象的资源
        delete[] data;
    
        // 3. 接管原对象的资源
        len = other.len;
        data = other.data;
    
        // 4. 原对象置空
        other.len = 0;
        other.data = nullptr;
    
        return *this;
    }
};

// 使用拷贝赋值运算符
MyString s1("hello");
MyString s2("world");
s2 = s1; // 调用拷贝赋值，s2 释放原有资源，深拷贝 s1 的资源


// 使用移动赋值运算符
MyString s1("hello");
MyString s2;
s2 = std::move(s1); // 调用移动赋值，s2 接管 s1 的资源

```

# 坑！
1. 如果没有手动定义，编译器会默认生成拷贝构造函数（浅拷贝）、拷贝赋值运算符（浅拷贝）。
2. C++11 后，如果没有手动定义拷贝 / 移动相关函数，编译器会默认生成移动构造 / 移动赋值，但如果手动定义了拷贝构造 / 拷贝赋值，默认移动函数会被禁用。
3. 如果手动定义了移动构造函数，编译器会禁止自动生成拷贝构造函数。The implicitly-declared copy constructor for class T is defined as deleted if T declares a move constructor or move assignment operator.



# 感悟
C++ 从 C 继承了 “值语义”（变量赋值 / 传参就是复制数据），为了保持这种直觉式的行为，默认给所有类生成浅拷贝构造函数，如果编译器不默认生成拷贝构造，开发者需要为每个简单类手动写空的拷贝构造，这会导致大量冗余代码。在 C++11 引入移动语义前，“拷贝” 是对象复制的唯一方式。