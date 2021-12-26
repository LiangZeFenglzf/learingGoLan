---
# 第三章　基础数据类型

虽然从底层而言，所有的数据都是由比特组成，但计算机一般操作的是固定大小的数，如整数、浮点数、比特数组、内存地址等。Go语言提供了丰富的数据组织形式，这依赖于Go语言内置的数据类型。

Go语言将数据类型分为四类：基础类型、复合类型、引用类型和接口类型。引用类型包括指针、切片、字典（§4.3）、函数（§5）、通道（§8），虽然数据种类很多，但它们都是对程序中一个变量或状态的间接引用。这意味着对任一引用类型数据的修改都会影响所有该引用的拷贝。 
---

[TOC]



# 第四章　复合数据类型

在第三章我们讨论了基本数据类型，它们可以用于构建程序中数据的结构，是Go语言世界的原子。在本章，我们将讨论复合数据类型，它是以不同的方式组合基本类型而构造出来的复合数据类型。讨论四种类型——数组、slice、map和结构体

数组和结构体是聚合类型；它们的值由许多元素或成员字段的值组成。数组是由同构的元素组成——每个数组元素都是完全相同的类型——结构体则是由异构的元素组成的。数组和结构体都是有固定内存大小的数据结构。相比之下，slice和map则是动态的数据结构，它们将根据需要动态增长。

## 收获 

数组就是一个僵化的东西。但是数组不像字符串，数组元素可以通过指针修改。数组当作函数传入参数时，是值拷贝的方式。也就意味着 传入一个超长的数组带来的开销会非常大，需要重新开辟内存，供 值与视为传入参数的数组的值 一致的·新数组 使用。

## 疑惑

数组 可以通过函数 传入引用数组的指针来修改 数组元素。

传入数组 是不会通过函数修改数组元素的。【值拷贝】

那 为什么 字符串没有 传入指针 修改字符串值的说法？ 字符串不能取址? unaddressable？

相信看了指针会有所收获。

## 数组

https://go.dev/ref/spec#Array_types

#### Array types

An array is a numbered sequence of elements of a single type, called the element type. The number of elements is called the length of the array and *is never negative.数组长度非负*

```
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

The length is part of the array's type; 

长度是数组类型的一部分

it must evaluate to a non-negative [constant](https://go.dev/ref/spec#Constants) [representable](https://go.dev/ref/spec#Representability) by a value of type `int`.
值必须为一个非负的常量，可由类型为' int '的值表示。

 The length of array `a` can be discovered using the built-in function [`len`](https://go.dev/ref/spec#Length_and_capacity). 

数组长度 可有built-in 下的len函数求得

The elements can be addressed by integer [indices](https://go.dev/ref/spec#Index_expressions) 0 through `len(a)-1`. Array types are always one-dimensional but may be composed to form multi-dimensional types.元素可通过索引 0~len-1。 数组类型总数一维，可以被组合成多维度

```
[32]byte
[2*N] struct { x, y int32 }
[1000]*float64
[3][5]int
[2][2][2]float64  // same as [2]([2]([2]float64))
```

#### 什么是数组

数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。因为数组的长度是固定的，因此在Go语言中很少直接使用数组。和数组对应的类型是Slice（切片），它是可以增长和收缩的动态序列，slice功能也更灵活，但是要理解slice工作原理的话需要先理解数组。

#### 数组索引

数组的每个元素可以通过索引下标来访问，索引下标的范围是从0开始到数组长度减1的位置。内置的len函数将返回数组中元素的个数。

默认情况下，数组的每个元素都被初始化为元素类型对应的零值，对于数字类型来说就是0。我们也可以使用数组字面值语法用一组值来初始化数组：

```Go
var q [3]int = [3]int{1, 2, 3}
```

在数组字面值中，如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算。因此，上面q数组的定义可以简化为

```Go
q := [...]int{1, 2, 3}
fmt.Printf("%T\n", q) // "[3]int"
```

#### 数组类型

数组的长度是数组类型的一个组成部分，因此[3]int和[4]int是两种不同的数组类型。数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定。

我们将会发现，数组、slice、map和结构体字面值的写法都很相似。上面的形式是直接提供顺序初始化值序列，但是也可以指定一个索引和对应值列表的方式初始化，就像下面这样：

```Go
type Currency int

const (
    USD Currency = iota // 美元      0
    EUR                 // 欧元      1
    GBP                 // 英镑	   2
    RMB                 // 人民币	   3
)

symbol := [...]string{USD: "$", EUR: "€", GBP: "￡", RMB: "￥"}
symbol := [...]string{0: "$", 1: "€", 2: "￡", 3: "￥"}
fmt.Println(RMB, symbol[RMB]) // "3 ￥"
```

在这种形式的数组字面值形式中，初始化索引的顺序是无关紧要的，而且没用到的索引可以省略，和前面提到的规则一样，未指定初始值的元素将用零值初始化。例如，

```Go
r := [...]int{99: -1}
```

定义了一个含有100个元素的数组r，最后一个元素被初始化为-1，其它元素都是用0初始化。

如果一个数组的元素类型是可以相互比较的，那么数组类型也是可以相互比较的，这时候我们可以直接通过==比较运算符来比较两个数组，只有当两个数组的所有元素都是相等的时候数组才是相等的。不相等比较运算符!=遵循同样的规则。

作为一个真实的例子，crypto/sha256包的Sum256函数对一个任意的字节slice类型的数据生成一个对应的消息摘要。消息摘要有256bit大小，因此对应[32]byte数组类型。如果两个消息摘要是相同的，那么可以认为两个消息本身也是相同（译注：理论上有HASH码碰撞的情况，但是实际应用可以基本忽略）；如果消息摘要不同，那么消息本身必然也是不同的。下面的例子用SHA256算法分别生成“x”和“X”两个信息的摘要：

*gopl.io/ch4/sha256*

```Go
import "crypto/sha256"

func main() {
    c1 := sha256.Sum256([]byte("x"))
    c2 := sha256.Sum256([]byte("X"))
    fmt.Printf("%x\n %x\n %t\n %T\n", c1, c2, c1 == c2, c1)
    // Output:
    // 2d711642b726b04401627ca9fbac32f5c8530fb1903cc4db02258717921a4881
    // 4b68ab3847feda7d6c62c1fbcbeebfa35eab7351ed5e78f4ddadea5df64b8015
    // false
    // [32]uint8
}
```

上面例子中，两个消息虽然只有一个字符的差异，但是生成的消息摘要则几乎有一半的bit位是不相同的。

#### Go如何对待数组

当调用一个函数的时候，函数的每个调用参数将会被赋值给函数内部的参数变量，所以函数参数变量接收的是一个复制的副本，并不是原始调用的变量。因为函数参数传递的机制导致传递大的数组类型将是低效的，并且对数组参数的任何的修改都是发生在复制的数组上，并不能直接修改调用时原始的数组变量。

在这个方面，Go语言对待数组的方式和其它很多编程语言不同，其它编程语言可能会隐式地将数组作为引用或指针对象传入被调用的函数。

当然，我们可以显式地传入一个数组指针，那样的话函数通过指针对数组的任何修改都可以直接反馈到调用者。下面的函数用于给[32]byte类型的数组清零：

```Go
func zero(ptr *[32]byte) {
    for i := range ptr {
        ptr[i] = 0
    }
}
```

其实数组字面值[32]byte{}就可以生成一个32字节的数组。而且每个数组的元素都是零值初始化，也就是0。因此，我们可以将上面的zero函数写的更简洁一点：

```Go
func zero(ptr *[32]byte) {
    *ptr = [32]byte{}
}
```

虽然通过指针来传递数组参数是高效的，而且也允许在函数内部修改数组的值，**但是数组依然是僵化的类型，因为数组的类型包含了僵化的长度信息。**上面的zero函数并不能接收指向[16]byte类型数组的指针，而且也没有任何添加或删除数组元素的方法。由于这些原因，除了像SHA256这类需要处理特定大小数组的特例外，数组依然很少用作函数参数；相反，我们一般使用slice来替代数组。

