---
首先我们介绍New 和 make。分析异同。最后我们要结合所有内置的Go类型介绍关于zero值和初始化 
---

[TOC]



# 疑问

为什么make只针对slice map channel等引用类型？

只有引用类型  初始化和归0是 2个意思。而普通类型或者复合类型 归0 和初始化 都是给类型 规定好的0值

# New&Make

new和make之后都是可以立即使用的。

make只用于map slice channel

make 会 进行一个初始化 new只会置0

所以我们需要了解数据类型的置0值

# Allocation with `new`

## New返回的是指向类型为T的0值 的指针，指针类型*T

Go has two allocation primitives

[^primitives]: n. [计]基元（primitive 的复数）；原始事物；基本体

 the built-in functions `new` and `make`. They do different things and apply to different types, which can be confusing, but the rules are simple. Let's talk about `new` first. It's a built-in function that allocates memory, but unlike its name sakes in some other languages it does not *initialize* the memory, it only *zeros* it. That is, `new(T)` allocates zeroed storage for a new item of type `T` and returns its address, a value of type `*T`. In Go terminology, it returns a pointer to a newly allocated zero value of type `T`.

Go有两种分配原语，内置函数new和内置函数make。这两个做不同的事情，适用于不同的type。...先讨论New。New分配内存，但是不会初始化内存，只会将其*归零*

也就是说， `new(T)`会分配一个归zero的存储给T类型的item【这里可以和数据结构里面对数据的描述挂钩data item】，返回地址，也就是一个*T类型的值。

在 Go 术语中，它返回一个指向 新分配的类型为T的0值 的    指针 。

这意味着数据结构的用户可以创建一个`new`并开始工作。例如，文档`bytes.Buffer`说明“零值`Buffer`是一个准备使用的空缓冲区”。同样，`sync.Mutex`没有显式构造函数或`Init`方法。相反，a 的零值`sync.Mutex` 被定义为未锁定的互斥锁。

零值有用的属性可以传递。考虑这个类型声明。

```
类型同步缓冲区结构{
    锁同步.互斥锁
    缓冲区字节.Buffer
}
```

type 的值`SyncedBuffer`也可以在分配或声明后立即使用。在下一个代码段中，`p`和`v`无需进一步安排即可正常工作。

Since the memory returned by `new` is zeroed, it's helpful to arrange when designing your data structures that the zero value of each type can be used without further initialization. 

由于通过`new`返回的存储空间已归零，因此在设计数据结构时安排每种类型的零值无需进一步初始化即可使用       是有帮助的。

This means a user of the data structure can create one with `new` and get right to work. 意味着通过内置函数new就可以立马使用不用初始化

举例：

For example, the documentation for `bytes.Buffer` states that "the zero value for `Buffer` is an empty buffer ready to use." 

[type Buffer]: https://pkg.go.dev/bytes@go1.17.5#Buffer

Similarly, `sync.Mutex` does not have an explicit constructor or `Init` method. Instead, the zero value for a `sync.Mutex` is defined to be an unlocked mutex.

[type mutex]: https://pkg.go.dev/sync@go1.17.5#Mutex

### 0值可用的传递性

The zero-value-is-useful property works transitively这个属性可以传递. 

Consider this type declaration.

一个复合结构，里面成员都是0值可用的成员

```
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

Values of type `SyncedBuffer` are also ready to use immediately upon allocation or just declaration. In the next snippet, both `p` and `v` will work correctly without further arrangement.一旦分配内存或刚刚声明，就可以使用。

```
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

# Allocation with `make`

Back to allocation. The built-in function `make(T, `*args*`)` serves a purpose different from `new(T)`.

首先make和new目的不同

## Make返回初始化的值，而不是New 归0而不初始化

### 适用数据类型slice,map,channel

 It creates slices, maps, and channels only, and it returns an *initialized* (not *zeroed*) value of type `T` (not `*T`). 

The reason for the distinction is that these three types represent, under the covers, references to data structures that must be initialized before use.

under the covers是插入成分，可不译。

造成差异的原因是，以上3中类型 代表的是引用，对 在使用之前必须初始化的数据结构的引用。

[^正常语序为]:  under the covers,the reason for the distinction is that these three types represent, references to data structures that must be initialized before use.

### Slice为例(先初始化后使用而不是right now use）

 A slice, for example, is a three-item descriptor containing a pointer to the data (inside an array), the length, and the capacity, and until those items are initialized, the slice is `nil`. 切片这一数据结构 是 包含3个数据项的描述。这3个data item分别是 指向数据的指针，长度，容量。在这3项初始化之前，切片是一个nil值

For slices, maps, and channels, `make` initializes the internal data structure and prepares the value for use. 

make内置函数会初始化内部的数据结构和准备好值提供使用。也即是make之后slice不为Nil 可以使用。

##### Slice分配的是 一个数组内存空间

For instance,

```
make([]int, 10, 100)
```



allocates an array of 100 ints and then creates a slice structure with length 10 and a capacity of 100 pointing at the first 10 elements of the array. 

分配数组内存空间，创建切片结构：长度10容量100，指针指向数组前10个元素

(When making a slice, the capacity can be omitted; see the section on slices for more information.) 

##### 重点：对比new内置函数

In contrast, `new([]int)` returns a pointer to a newly allocated, zeroed slice structure, that is, a pointer to a `nil` slice value.

对比new返回 一个指针 指向分配的归0的切片结构，也就是说 这个指针是引用Nil 切片值。

These examples illustrate the difference between `new` and `make`.

```
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

Remember that `make` applies only to maps, slices and channels and does not return a pointer. To obtain an explicit pointer allocate with `new` or take the address of a variable explicitly.

如果想要获得指针，使用New或者是直接获得变量地址 &取地址符

# Program initialization and execution

## The zero value

When storage is allocated for a **[variable](https://go.dev/ref/spec#Variables)**, either through a declaration or a call of `new`, or when a new **value** is created, either through a composite literal or a call of `make`, and no explicit initialization is provided, the variable or value is given a default value.

不论是 通过 声明|New调用 来为变量分配内存，还是 通过复合字面量|make调用 来创建值，如果没有提供明显的初始化，变量|值 会被给一个默认值

### 疑问

variable 和value的概念

### 所有定义类型的zero value

 Each element of such a variable or value is set to the *zero value* for its type:如表格

This initialization is done recursively, so for instance each element of an array of structs will have its fields zeroed if no value is specified.

初始化是递归，所以 结构体数组 的元素  （也就是 第i个结构体变量）会将字段 归0 如果没有指定

| Defined Type | zero value |
| ------------ | ---------- |
| booleans     | false      |
| numeric type | 0          |
| strings      | ""         |
| pointers     | nil        |
| functions    | nil        |
| interfaces   | nil        |
| slices       | nil        |
| channels     | nil        |
| maps         | nil        |

These two simple **declarations** are equivalent:

```
var i int
var i int = 0
```

After

```
type T struct { i int; f float64; next *T }
t := new(T)
```

the following holds:

```
t.i == 0
t.f == 0.0
t.next == nil  指针置0就是nil
```

The same would also be true after 

```
var t T
```

## Package initialization

暂不作说明在这份文档中

## Program execution

A complete program is created by linking a single, unimported package called the *main package* with all the packages it imports, transitively. The main package must have package name `main` and declare a function `main` that takes no arguments and returns no value.

一个完整程序由  唯一个 不能导出的称为main的包，main包带着所有导入的包 传递 着创建。

main包必须有  包名main ，声明一个不需要传入变量不带返回值的main函数   其实就是一个空函数签名

```
func main() { … }
```

Program execution begins by initializing the main package and then invoking the function `main`. When that function invocation returns, the program exits. It does not wait for other (non-`main`) goroutines to complete.

程序执行首先初始化主包，然后调用函数`main`。