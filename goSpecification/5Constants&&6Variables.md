[TOC]



# Constants

## 常量

### 6种常量，4种数值型常量

| *floating-point constants* | *rune constants* | integer constants | *complex constants* | boolean constants* | *string constants* |
| -------------------------- | ---------------- | ----------------- | ------------------- | ------------------ | ------------------ |

| numeric      constants | numeric constants | numeric constants | numeric constants | non-numeric | non-numeric constants |
| ---------------------- | ----------------- | ----------------- | ----------------- | ----------- | --------------------- |

### 常量值的表示方式

A constant value is represented by a [rune](https://go.dev/ref/spec#Rune_literals), [integer](https://go.dev/ref/spec#Integer_literals), [floating-point](https://go.dev/ref/spec#Floating-point_literals), [imaginary](https://go.dev/ref/spec#Imaginary_literals), or [string](https://go.dev/ref/spec#String_literals) literal, an identifier denoting a constant, a [constant expression](https://go.dev/ref/spec#Constant_expressions), a [conversion](https://go.dev/ref/spec#Conversions) with a result that is a constant,      or the result value of some built-in functions such as `unsafe.Sizeof` applied to any value, `cap` or `len` applied to [some expressions](https://go.dev/ref/spec#Length_and_capacity), `real` and `imag` applied to a complex constant and `complex` applied to numeric constants.  

The boolean truth values`布尔真值` are represented by the predeclared constants `true` and `false`. The predeclared identifier [iota](https://go.dev/ref/spec#Iota) denotes an integer constant.

常量值由符文、整数、浮点数、虚数或字符串文字、表示常量的标识符、常量表达式、结果为常量的转换或某些内置函数(如 unsafe)的结果值表示。

内置函数：1适用于任何值的`unsafe.Sizeof`，2适用于某些表达式的`cap` 或 `len` ，3适用于复杂的常量得的`real` 和 `imag `  ，4适用于数字常量的`complex`。

In general, `complex constants `are a form of [constant expression](https://go.dev/ref/spec#Constant_expressions) and are discussed in that section.

我们认为，`复数`常量是常数表达式的一种形式，本节将对其进行讨论。

#### Numeric constants数值常量可以表示的值

Numeric constants represent exact values of arbitrary precision and do not overflow. Consequently, there are no constants denoting the IEEE-754 negative zero, infinity, and not-a-number values.

数值常量表示任意精度的精确值，并且不会溢出。因此，没有常量可以表示 `IEEE-754` 负零、`无穷`和`非数值`。

### Typed|Untyped constant

Constants may be [typed](https://go.dev/ref/spec#Types) or *untyped*. Literal constants, `true`, `false`, `iota`, and certain [constant expressions](https://go.dev/ref/spec#Constant_expressions) containing only untyped constant operands are untyped.

常量可以是类型化的，也可以是非类型化的。字面量常量、 true、 false、 iota 和某些只包含非类型常量操作数的常量表达式是`无类型`的。

#### typed

A constant may be given a type explicitly by a [constant declaration](https://go.dev/ref/spec#Constant_declarations) or [conversion](https://go.dev/ref/spec#Conversions), or implicitly when used in a [variable declaration](https://go.dev/ref/spec#Variable_declarations) or an [assignment](https://go.dev/ref/spec#Assignments) or as an operand in an [expression](https://go.dev/ref/spec#Expressions). It is an error if the constant value cannot be [represented](https://go.dev/ref/spec#Representability) as a value of the` respective`相应 type.

常量可以通过常量声明或转换显式地给定类型，或者在变量声明或赋值中使用时隐式地给定类型，或者在表达式中作为操作数使用。如果常量值不能表示为相应类型的值，则为错误。

#### untyped

An untyped constant has a *default type* which is the type to which the constant is implicitly converted in contexts where a typed value is required, for instance, in a [short variable declaration](https://go.dev/ref/spec#Short_variable_declarations) such as `i := 0` where there is no explicit type. The default type of an untyped constant is `bool`, `rune`, `int`, `float64`, `complex128` or `string` respectively, depending on whether it is a boolean, rune, integer, floating-point, complex, or string constant.

非类型化常量具有默认类型，该类型是在需要`typed`值的上下文中隐式转换该常量的类型，例如，在没有显式类型的简短变量声明中，如 i: = 0。`typed`常量的默认类型分别为 `bool`、 `rune`、` int`、 `float64`、` complex128`或` string`，具体取决于它是布尔型、 rune 型、整数型、浮点型、复数型还是字符串`constant`。

Implementation restriction: Although numeric constants have arbitrary precision in the language, a compiler may implement them using an internal representation with limited precision. That said, every implementation must:

实现限制: 尽管数值常量在语言中具有任意精度，但编译器可以使用有限精度的内部表示来实现它们。就是说，每一项实施都必须:

- Represent integer constants with at least 256 bits.

  用至少256位表示整数常量。

- Represent floating-point constants, including the parts of a complex constant, with a mantissa of at least 256 bits and a signed binary exponent of at least 16 bits.

  表示浮点常数，包括复常数的部分，尾数至少为256位，有符号二进制指数至少为16位。

- Give an error if unable to represent an integer constant precisely.

  如果无法精确表示整数常数，则给出错误。

- Give an error if unable to represent a floating-point or complex constant due to overflow.

  如果由于溢出而无法表示浮点或复常数，则给出错误。

- Round to the nearest representable constant if unable to represent a floating-point or complex constant due to limits on precision.

  如果由于精度限制而无法表示浮点数或复数常数，则舍入到最接近的可表示常数。

These requirements apply both to literal constants and to the result of evaluating [constant expressions](https://go.dev/ref/spec#Constant_expressions).

这些要求既适用于字面量常量，也适用于计算常量表达式的结果。

# Variables

## 变量

A variable is a storage location for holding a *value*. The set of permissible values is determined by the variable's *[type](https://go.dev/ref/spec#Types)*.

变量是保存值的存储位置。允许值的集合由变量的类型确定。

### 什么时候会给变量分配存储空间storage

A [variable declaration](https://go.dev/ref/spec#Variable_declarations) or, for function parameters and results, the signature of a [function declaration](https://go.dev/ref/spec#Function_declarations) or [function literal](https://go.dev/ref/spec#Function_literals)                 

reserves `storage` for a named variable. Calling the built-in function [`new`](https://go.dev/ref/spec#Allocation) or taking the address of a [composite literal](https://go.dev/ref/spec#Composite_literals) allocates `storage `for a variable at run time. Such an anonymous variable is referred to via a (possibly implicit) [pointer indirection](https://go.dev/ref/spec#Address_operators).

`1变量声明`，或者，对于函数参数和结果，`2函数声明的签名`或者`3函数字面值`    为   指定的        变量      ` 保留`存储空间。 

在运行时调用内置函数 new 或获取复合字面量的地址为变量`分配`存储空间。这样的匿名变量通过一个(可能是隐式的)指针间接引用。

### 结构化变量的元素变量

*Structured* variables of [array](https://go.dev/ref/spec#Array_types), [slice](https://go.dev/ref/spec#Slice_types), and [struct](https://go.dev/ref/spec#Struct_types) types have elements and fields that may be [addressed](https://go.dev/ref/spec#Address_operators) individually. Each such element acts like a variable.

数组、片和结构类型的结构化变量具有可单独寻址的元素和字段。每个这样的元素都像一个变量。

#### static type定义

The *static type* (or just *type*) of a variable is the type given in its declaration, the type provided in the `new` call or composite literal, or the type of an element of a structured variable. 变量的静态类型(或仅仅类型)是其声明中给出的类型、`new`调用或复合字面量中提供的类型 或结构化变量的元素 类型。

#### dynamic type:interface

Variables of interface type also have a distinct *dynamic type*, which is the concrete type of the value assigned to the variable at run time (unless the value is the predeclared identifier `nil`, which has no type). The dynamic type may vary during execution but values stored in interface variables are always [assignable](https://go.dev/ref/spec#Assignability) to the static type of the variable.

接口类型的变量是一个不同的动态类型，这是在运行时分配给变量的值的具体类型(除非该值是预声明的标识符 nil，它没有类型)。在执行过程中，动态类型可能会发生变化，但是存储在接口变量中的值总是可以分配给变量的静态类型。

```
var x interface{}  // x is nil and has static type interface{}
var v *T           // v has value nil, static type *T
x = 42             // x has value 42 and `dynamic` type int
x = v              // x has value (*T)(nil) and `dynamic` type *T
```

A variable's value is retrieved by referring to the variable in an [expression](https://go.dev/ref/spec#Expressions); it is the most `recent` value [assigned](https://go.dev/ref/spec#Assignments) to the variable【因为接口的动态类型运行时可变就算是`zeroed value`也会变化】. If a variable has not yet been assigned a value, its value is the [zero value](https://go.dev/ref/spec#The_zero_value) for its type.

通过引用表达式中的变量来检索变量的值; 它是分配给该变量的最新值。如果一个变量还没有被赋值，那么它的值就是它的类型的零值。