---
是constant和variable的区别
变量 说法 是 赋值  因为变
常量 说法 是 表示  因为不变
先有可表示性再有可赋值性
---



[TOC]



# Properties of types and values

## 类型和值的属性

### 1Type identity类型标识

A [defined type](https://go.dev/ref/spec#Type_definitions) is always different from any other type. Otherwise, two types are identical if their [underlying](https://go.dev/ref/spec#Types) type literals are structurally equivalent; that is, they have the same literal structure and corresponding components have identical types. In detail:

已定义的类型始终不同于任何其他类型。其他情况下，如果两个类型的基础类型字面量在结构上是等价的，那么它们是相同的; 也就是说，它们具有相同的字面量结构，而相应的组件具有相同的类型。详细内容:

- Two array types are identical if they have identical element types and the same array length.

  如果两个数组具有相同的元素类型和相同的数组长度，那么它们是相同的。

- Two slice types are identical if they have identical element types.

  如果两个切片类型具有相同的元素类型，那么它们是相同的。

- Two struct types are identical if they have the same sequence of fields, and if corresponding fields have the same names, and identical types, and identical tags. [Non-exported](https://go.dev/ref/spec#Exported_identifiers) field names from different packages are always different.

  如果两个结构类型具有相同的字段序列，并且相应的字段具有相同的名称、相同的类型和相同的标记，则它们是相同的。来自不同包的非导出字段名称总是不同的。

- Two pointer types are identical if they have identical base types.

  如果两个指针类型具有相同的基类型，则它们是相同的。

- Two function types are identical if they have the same number of parameters and result values, corresponding parameter and result types are identical, and either both functions are variadic or neither is. Parameter and result names are not required to match.

  如果两个函数类型具有相同数目的参数和结果值，那么它们是相同的，相应的参数和结果类型是相同的，并且两个函数要么是可变参数，要么都不是。参数和结果名称不需要匹配。

- Two interface types are identical if they have the same set of methods with the same names and identical function types. [Non-exported](https://go.dev/ref/spec#Exported_identifiers) method names from different packages are always different. The order of the methods is irrelevant.

  如果两个接口类型具有相同的方法集，且具有相同的名称和相同的函数类型，则它们是相同的。来自不同包的非导出方法名称总是不同的。方法的顺序无关紧要。

- Two map types are identical if they have identical key and element types.

  如果两个映射类型具有相同的键和元素类型，那么它们是相同的。

- Two channel types are identical if they have identical element types and the same direction.

  如果两个通道具有相同的元素类型和相同的方向，则它们是相同的。

#### 考虑类型声明的两种方式TypeDef &&AliasDecl

Given the declarations

考虑到这些声明

```
type (
	A0 = []string
	A1 = A0
	A2 = struct{ a, b int }
	A3 = int
	A4 = func(A3, float64) *A0
	A5 = func(x int, _ float64) *[]string
)

type (
	B0 A0
	B1 []string
	B2 struct{ a, b int }
	B3 struct{ a, c int }
	B4 func(int, float64) *B0
	B5 func(x int, y float64) *A1
)

type	C0 = B0
```

these types are identical:

这些类型是一样的:

```
A0, A1, and []string
A2 and struct{ a, b int }
A3 and int
A4, func(int, float64) *[]string, and A5

B0 and C0
[]int and []int
struct{ a, b *T5 } and struct{ a, b *T5 }
func(x int, y float64) *[]string, func(int, float64) (result *[]string), and A5
```

#### TypeDef出来就是不同的type;别名声明 可以是相同type

`B0` and `B1` are different because they are new types created by distinct [type definitions](https://go.dev/ref/spec#Type_definitions); `func(int, float64) *B0` and `func(x int, y float64) *[]string` are different because `B0` is different from `[]string`.

B0和 b1是不同的，因为它们是由不同的类型定义创建的新类型; func (int，float64) * b0和 func (x int，y float64) * [] string 是不同的，因为 b0不同于[] string。

### 2Assignability可赋值性：值可赋值给 变量

A value `x` is *assignable* to a [variable](https://go.dev/ref/spec#Variables) of type `T` ("`x` is assignable to `T`") if one of the following conditions applies:

如果下列条件之一适用，则值 x 可分配给类型为 t 的变量(“ x 可分配给 t”) :

- `x`'s type is identical to `T`.         X 的类型与 t 相同。

- `x`'s type `V` and `T` have identical [underlying types](https://go.dev/ref/spec#Types) and at least one of `V` or `T` is not a [defined](https://go.dev/ref/spec#Type_definitions) type.X 的类型 v 和 t 具有相同的基础类型，而且 v 或 t 中至少有一个类型不是已定义类型。【如果两个都是TypeDef来，需要显式类型转换】

  ##### 接口的动态类型

- `T` is an interface type and `x` [implements](https://go.dev/ref/spec#Interface_types) `T`. 是一种接口类型，x 实现了 t。

- `x` is a bidirectional channel value, `T` is a channel type, `x`'s type `V` and `T` have identical element types, and at least one of `V` or `T` is not a defined type.

  X 是一个双向通道值，t 是一个通道类型，x 的类型 v 和 t 有相同的元素类型，并且至少有一个 v 或 t 不是已定义的类型。

- `x` is the predeclared identifier `nil` and `T` is a pointer, function, slice, map, channel, or interface type.

  X 是预声明的标识符 nil，t 是指针、函数、片、映射、通道或接口类型。

  ##### 参考6Varables.md

- `x` is an untyped [constant](https://go.dev/ref/spec#Constants) [representable](https://go.dev/ref/spec#Representability) by a value of type `T`.

  X 是一个无类型的常数，用 t 类型的值表示。

### 3Representability可表示性：常量由类型T表示

A [constant](https://go.dev/ref/spec#Constants) `x` is *representable* by a value of type `T` if one of the following conditions applies:

常量 x 可以用类型 t 的值表示，如果下列条件适用:

- `x` is in the set of values [determined](https://go.dev/ref/spec#Types) by `T`.

  X 在 t 确定的值集中。

- `T` is a floating-point type and `x` can be rounded to `T`'s precision without overflow. Rounding uses IEEE 754 round-to-even rules but with an IEEE negative zero further simplified to an unsigned zero. `Note` that constant values never result in an IEEE negative zero, NaN, or infinity.

  T 是浮点类型，x 可以舍入到 t 的精度而不会溢出。四舍五入使用 IEEE 754四舍五入规则，但使用 IEEE 负零进一步简化为无符号零。请注意，常量值不会导致 IEEE 负零、 NaN 或无穷大。

- `T` is a complex type, and `x`'s [components](https://go.dev/ref/spec#Complex_numbers) `real(x)` and `imag(x)` are representable by values of `T`'s component type (`float32` or `float64`).

  T 是一个`复数`类型，x 的分量 real (x)和 imag (x)可以通过 t 的分量类型(float32或 float64)的值来表示。

​      **x                                       T                       x is representable by a value of T because**

```
'a'                 byte        97 is in the set of byte values
97                  rune        rune is an alias for int32, and 97 is in the set of 32-bit integers
"foo"               string      "foo" is in the set of string values
1024                int16       1024 is in the set of 16-bit integers
42.0                byte        42 is in the set of unsigned 8-bit integers
1e10                uint64      10000000000 is in the set of unsigned 64-bit integers
2.718281828459045   float32     2.718281828459045 rounds to 2.7182817 which is in the set of float32 values
-1e-1000            float64     -1e-1000 rounds to IEEE -0.0 which is further simplified to 0.0
0i                  int         0 is an integer value
(42 + 0i)           float32     42.0 (with zero imaginary part) is in the set of float32 values
```

**x                                            T                     x is not representable by a value of T because**

```
0                   bool        0 is not in the set of boolean values                        false|true
'a'                 string      'a' is a rune, it is not in the set of string values          ` ` | ""
1024                byte   ==int8   2<sup>7</sup>-1 =127  1024 is not in the set of unsigned 8-bit integers 
-1                  uint16      -1 is not in the set of unsigned 16-bit integers     2<sup>16</sup>-1
1.1                 int         1.1 is not an integer value
42i                 float32     (0 + 42i) is not in the set of float32 values
1e1000              float64     1e1000 overflows to IEEE +Inf after rounding


```

