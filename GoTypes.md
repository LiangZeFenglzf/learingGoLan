---
介绍各种类型
---

[TOC]



# Types

## 1.1类型是一个什么样的概念

A type determines a set of values together with operations and methods specific to those values.

类型确定一组值以及特定于这些值的操作和方法。

## 1.2类型字面量概念

 A type may be denoted by a *type name*【1.3提到的type names】, if it has one, or specified using a *type literal*, which composes a type from existing types.

一个类型可以用一个*类型名称*来表示（如果有这样一个类型名），或者使用一个*类型字面量来*指定（类型字面量：从现有的类型组成一个类型。）

```
Type      = TypeName | TypeLit | "(" Type ")" .
            3选一                 第3个是什么意思
TypeName  = identifier | QualifiedIdent .
			标识符【参考1.3】 或 合法标识符
TypeLit   = ArrayType | StructType | PointerType | FunctionType | InterfaceType |
	    SliceType | MapType | ChannelType .
            TypeLit是TypeLiteral，也就是类型字面量。
```

### 1.2.1注意模糊点

从1.3得知 ArrayType不是我们说的array。

TypeName 只有boolean ，numeric type ，string type

## 1.3预声明类型

The language [predeclares](https://go.dev/ref/spec#Predeclared_identifiers) certain **type names**.

```
预声明标识符Predeclared identifiers
The following identifiers are implicitly declared in the universe block:
                                           以下标识符 都是暗自声明 在全局范围内
预声明类型Types:
	bool byte complex64 complex128 error float32 float64
	int int8 int16 int32 int64 rune string
	uint uint8 uint16 uint32 uint64 uintptr
      
预声明常量Constants:
	true false iota

预声明0值Zero value:
	nil

预声明函数Functions:
	append cap close complex copy delete imag len
	make new panic print println real recover
```

 Others are introduced with [type declarations](https://go.dev/ref/spec#Type_declarations). Go语言[预先声明了](https://go.dev/ref/spec#Predeclared_identifiers)某些类型名称。其他的则是通过[类型声明](https://go.dev/ref/spec#Type_declarations)引入的。

*Composite types*—array, struct, pointer, function, interface, slice, map, and channel types—may be constructed using type literals.

*复合类型——*数组、结构、指针、函数、接口、切片、映射和通道类型——可以使用TypeLit构造。

## 1.4基类判断

Each type `T` has an *underlying type*: 

### 1.4.1规则一：要清楚TypeLit的定义1.2

If `T` is one of the predeclared boolean, numeric, or string types, or a type literal, the corresponding underlying type is `T` itself. 

如果`T` 是预先声明的布尔值、数字或字符串类型之一，或者TypeLit，则相应的底层类型是`T`自己。

### 1.4.2规则二：

Otherwise, `T`'s underlying type is the underlying type of the type to which `T` refers in its [type declaration](https://go.dev/ref/spec#Type_declarations).

否则，`T`的基础类型是`T`在其TypeDef中引用 的类型的基础类型。【这里参考TypeDecl文档，类型声明有2种方式】

 如果引用的类型也不是预声明类型

```
type (
	A1 = string   类型定义的别名定义    A1的基类是string
	A2 = A1                         A2的基类是A1，而A1... 所以A2基类是String
)

type (
	B1 string                         string
	B2 B1                             string
	B3 []B1                         
	B4 B3
)
```

#### 重点来喽：

B3在TypeDecl语句中 refer的是 []B1   由于[]T是ArrayType 也就是对应1.2里面的TypeLit也即是对应规则一 .

B3的基类根据规则1  是                                       []B1

B4参照规则2是B3然后B3的基类是[]B1最终

B4的基类也是                                                       []B1

而不是我们想的参照规则2  变成                       []string



## 2介绍各种类型

### 2.1Method sets

A type has a (possibly empty) *method set* associated with it. 对应上类型的定义1.1 操作

The method set of an [interface type](https://go.dev/ref/spec#Interface_types) is its interface. 

The method set of any other type `T` consists of all [methods](https://go.dev/ref/spec#Method_declarations) declared with receiver type `T`. 

只要类型为T ,方法集 包含了 所有 声明了接收者类型为T的方法。

The method set of the corresponding [pointer type](https://go.dev/ref/spec#Pointer_types) `*T` is the set of all methods declared with receiver `*T` or `T` (that is, it also contains the method set of `T`).

类型为*T的包含了 声明接收者类型为T和\n *T的方法

 Further rules apply to structs containing embedded fields, as described in the section on [struct types](https://go.dev/ref/spec#Struct_types). 更多规则适用于包含嵌入字段的结构，如[结构类型](https://go.dev/ref/spec#Struct_types)部分所述。

Any other type has an empty method set. In a method set, each method must have a [unique](https://go.dev/ref/spec#Uniqueness_of_identifiers) non-[blank](https://go.dev/ref/spec#Blank_identifier) [method name](https://go.dev/ref/spec#MethodName).任何其他类型都有一个空的方法集。在一个方法集中，每个方法.....

#### 2.1.1不懂

The method set of a type determines the interfaces that the type [implements](https://go.dev/ref/spec#Interface_types) and the methods that can be [called](https://go.dev/ref/spec#Calls) using a receiver of that type.

类型的方法集决定了该类型[实现](https://go.dev/ref/spec#Interface_types)的接口 以及可以 使用该类型的接收器[调用](https://go.dev/ref/spec#Calls)的方法。

### 2.2Boolean types

A *boolean type* represents the set of Boolean truth values denoted by the predeclared constants `true` and `false`. The predeclared boolean type is `bool`; it is a [defined type](https://go.dev/ref/spec#Type_definitions).这个类型 就是代表了 预声明常量 true |false 这两个真值。

### 2.3Numeric types

A *numeric type* represents sets of integer or floating-point values. The predeclared architecture-independent numeric types are:

```
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts
complex128  the set of all complex numbers with float64 real and imaginary parts

byte        alias for uint8
rune        alias for int32
```

The value of an *n*-bit integer is *n* bits wide and represented using [two's complement arithmetic](https://en.wikipedia.org/wiki/Two's_complement).

n—bit整数类型的值是n bits 由 二进制补码表示

There is also a set of predeclared numeric types with implementation-specific sizes:

```
uint     either 32 or 64 bits
int      same size as uint
uintptr  an unsigned integer large enough to store the uninterpreted bits of a pointer value
```

To avoid portability issues all numeric types are [defined types](https://go.dev/ref/spec#Type_definitions) and thus distinct except `byte`, which is an [alias](https://go.dev/ref/spec#Alias_declarations) for `uint8`, and `rune`, which is an alias for `int32`. Explicit conversions are required when different numeric types are mixed in an expression or assignment. For instance, `int32` and `int` are not the same type even though they may have the same size on a particular architecture.

为了避免可移植性问题所有数值类型是defined types，

#### 2.3.1非defined types:byte&rune

是TypeDecl中的AliasDecl

除了 `byte`，`uint8`的别名； `rune`，`int32`别名。当表达式或赋值中混合不同的数字类型时，需要显式转换。例如，`int32`而`int` 不是即使他们可能对特定的架构相同尺寸相同的类型。

### 2.4String types

A *string type* represents the set of string values. A string value is a (possibly empty) sequence of bytes.

字符串值是 字节序列 可能是空的字节序列

 The number of bytes is called the length of the string and is never negative. 字节长度是字符串的长度，非负

Strings are immutable【不可变的】: once created, it is impossible to change the contents of a string. 一旦创建内容不可改变

The predeclared string type is `string`; it is a [defined type](https://go.dev/ref/spec#Type_definitions).

#### 2.4.1字符串操作 s[i]：取字符串的第i个字节

The length of a string `s` can be discovered using the built-in function [`len`](https://go.dev/ref/spec#Length_and_capacity). The length is a compile-time constant if the string is a constant. A string's bytes can be accessed by integer [indices](https://go.dev/ref/spec#Index_expressions) 0 through `len(s)-1`. 内置函数len返回 字符串长度。

字符串的字节序列可通过索引 0~len-1获得。

#### 2.4.12不可对s[i]取地址

It is illegal to take the address of such an element; if `s[i]` is the `i`'th byte of a string, `&s[i]` is invalid. 如果元素S[i]是 字符串第i个字节 那么 取地址是无效的。

### 2.5Array types

#### 2.5.1defintion

An array is a numbered sequence of elements of a single type, called the element type.

是以可数的，元素类型单一的序列。这个类型就就叫元素类型

 The number of elements is called the length of the array and is never negative.

```
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

##### length 规定

The length is part of the array's type; it must evaluate to a non-negative [constant](https://go.dev/ref/spec#Constants) [representable](https://go.dev/ref/spec#Representability) by a value of type `int`. 

长度也是数组类型定义的一部分。 它必须要是 一个 非负的 由一个int类型值 代表的constant

The length of array `a` can be discovered using the built-in function [`len`](https://go.dev/ref/spec#Length_and_capacity). 

#### 2.5.2可取地址 

The elements can be addressed by integer [indices](https://go.dev/ref/spec#Index_expressions) 0 through `len(a)-1`. Array types are always one-dimensional but may be composed to form multi-dimensional types.

```
[32]byte
[2*N] struct { x, y int32 }
[1000]*float64
[3][5]int
[2][2][2]float64  // same as [2]([2]([2]float64))
```

### 2.6Slice types

#### 2.6.1切片 3部分构成：指针 长度 容量

A slice is a descriptor for a contiguous segment of an *underlying array* and provides access to a numbered sequence of elements from that array. A slice type denotes the set of all slices of arrays of its element type. The number of elements is called the length of the slice and is never negative.

##### 不初始化的只有 zero value：nil

The value of an uninitialized slice is `nil`.

```
SliceType = "[" "]" ElementType .
```

The length of a slice `s` can be discovered by the built-in function [`len`](https://go.dev/ref/spec#Length_and_capacity); 

##### 切片长度可变，数组长度不变

unlike with arrays it may change during execution. 

##### 切片像 数组一样可 对元素取地址

The elements can be addressed by integer [indices](https://go.dev/ref/spec#Index_expressions) 0 through `len(s)-1`. 

##### 切片索引小于底层数组索引

The slice index of a given element may be less than the index of the same element in the underlying array.

#### 2.6.2切片共享内存，数组不共享

A slice, once initialized, is always associated with an underlying array that holds its elements. A slice therefore shares storage with its array and with other slices of the same array; by contrast, distinct arrays always represent distinct storage.

#### 2.6.3容量定义： 从切片第一个元素开始 到 其底层数组 最后一个元素 的Numbers

The array underlying a slice may extend past the end of the slice. The *capacity* is a measure of that extent: it is the sum of the length of the slice and the length of the array beyond the slice; a slice of length up to that capacity can be created by [*slicing*](https://go.dev/ref/spec#Slice_expressions) a new one from the original slice. The capacity of a slice `a` can be discovered using the built-in function [`cap(a)`](https://go.dev/ref/spec#Length_and_capacity).

#### 2.6.4make内置函数 初始化 slice slice非Nil

A new, initialized slice value for a given element type `T` is made using the built-in function [`make`](https://go.dev/ref/spec#Making_slices_maps_and_channels), which takes a slice type and parameters specifying the length and optionally the capacity. A slice created with `make` always allocates a new, hidden array to which the returned slice value refers. That is, executing

```
make([]T, length, capacity)
```

produces the same slice as allocating an array and [slicing](https://go.dev/ref/spec#Slice_expressions) it, so these two expressions are equivalent:

```
make([]int, 50, 100)
new([100]int)[0:50]
```

Like arrays, slices are always one-dimensional but may be composed to construct higher-dimensional objects. With arrays of arrays, the inner arrays are, by construction, always the same length; however with slices of slices (or arrays of slices), the inner lengths may vary dynamically. Moreover, the inner slices must be initialized individually.

### 2.7Struct types

#### 2.7.1结构体定义

A struct is a sequence of named elements, called fields, each of which has a name and a type. Field names may be specified explicitly (IdentifierList) or implicitly (EmbeddedField). 

结构体就是 一系列     被称为字段的  有名字的元素，每一个字段都有一个name 和type。字段名字要 用标识符显式指定，要么就是隐式（这种不明显指定名字的我们称之为插入的字段）

Within a struct, non-[blank](https://go.dev/ref/spec#Blank_identifier) field names must be [unique](https://go.dev/ref/spec#Uniqueness_of_identifiers).在一个结构体  非空的字段 名必须唯一

```
StructType    = "struct" "{" 
                    { FieldDecl ";" }     
                    
                  "}" .


FieldDecl     = (IdentifierList Type | EmbeddedField) [ Tag ] .
EmbeddedField = [ "*" ] TypeName .
Tag           = string_lit .
// An empty struct.
struct {}

// A struct with 6 fields.
struct {
	x, y int
	u float32
	_ float32  // padding
	A *[]int
	F func()
}
```

##### 插入字段的定义

A field declared with a type but no explicit field name is called an *embedded field*. An embedded field must be specified as a type name `T` or as a pointer to a non-interface type name `*T`, and `T` itself may not be a pointer type. 插入字段可以是T也可以是指针类型 但指针类型的基类不能是指针

The unqualified type name acts as the field name.不合法的类型名作为字段名

```
// A struct with four embedded fields of types T1, *T2, P.T3 and *P.T4
struct {
	T1        // field name is T1
	*T2       // field name is T2
	P.T3      // field name is T3
	*P.T4     // field name is T4
	x, y int  // field names are x and y
}
```

##### 非法字段名：非空字段名不唯一

The following declaration is illegal because field names must be unique in a struct type:

```
struct {
	T     // conflicts with embedded field *T and *P.T
	*T    // conflicts with embedded field T and *P.T
	*P.T  // conflicts with embedded field T and *T
}
```

##### 目前看不懂：promoted 什么意思,selector的概念不明白

A field or [method](https://go.dev/ref/spec#Method_declarations) `f` of an embedded field in a struct `x` is called *promoted* if `x.f` is a legal [selector](https://go.dev/ref/spec#Selectors) that denotes that field or method `f`.

Promoted fields act like ordinary fields of a struct except that they cannot be used as field names in [composite literals](https://go.dev/ref/spec#Composite_literals) of the struct.

#### 2.7.2方法集的继承come back to Types/Methods set

Given a struct type `S` and a [defined type](https://go.dev/ref/spec#Type_definitions) `T`, **promoted methods** are included in the method set of the struct as follows:

##### 内嵌字段是不是指针类型 ，指针类型 继承所有

- If `S` contains an embedded field `T`, the [method sets](https://go.dev/ref/spec#Method_sets) of `S` and `*S` both include promoted methods with receiver `T`. The method set of `*S` also includes promoted methods with receiver `*T`.
- If `S` contains an embedded field `*T`, the method sets of `S` and `*S` both include promoted methods with receiver `T` or `*T`.

##### 字段tag

A field declaration may be followed by an optional string literal *tag*, which becomes an attribute for all the fields in the corresponding field declaration. 

An empty tag string is equivalent to an absent tag. The tags are made visible through a [reflection interface](https://go.dev/pkg/reflect/#StructTag) and take part in [type identity](https://go.dev/ref/spec#Type_identity) for structs but are otherwise ignored.反射接口可见，参与type身份验证，其他情况直接忽略

```
struct {
	x, y float64 ""  // an empty tag string is like an absent tag
	name string  "any string is permitted as a tag"
	_    [4]byte "ceci n'est pas un champ de structure"
}

// A struct corresponding to a TimeStamp protocol buffer.
// The tag strings define the protocol buffer field numbers;
// they follow the convention outlined by the reflect package.
struct {
	microsec  uint64 `protobuf:"1"`
	serverIP6 uint64 `protobuf:"2"`
}
```

### 2.8Pointer types

A pointer type denotes the set of all pointers to [variables](https://go.dev/ref/spec#Variables) of a given type, called the *base type* of the pointer. 

##### 2.8.1特点：没有初始化的指针值为nil；联系切片

The value of an uninitialized pointer is `nil`.

```
PointerType = "*" BaseType .
BaseType    = Type .
*Point
*[4]int
```

### 2.9Function types

A function type denotes the set of all functions with the same parameter and result types. 

##### 2.9.1特点：未初始化的值为nil

The value of an uninitialized variable of function type is `nil`.

```
FunctionType   = "func" Signature .    函数签名是除func的剩下的·部分，除去函数体
Signature      = Parameters [ Result ] .   结果列表是可选的。
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .   参数列表是可选的 也就是空参函数
ParameterList  = ParameterDecl { "," ParameterDecl } .  多个参数声明
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
```

Within a list of parameters or results, the names (IdentifierList) must either all be present or all be absent. If present, each name stands for one item (parameter or result) of the specified type and all non-[blank](https://go.dev/ref/spec#Blank_identifier) names in the signature must be [unique](https://go.dev/ref/spec#Uniqueness_of_identifiers). If absent, each type stands for one item of that type. 

在参数或结果列表中，名称 (IdentifierList) 必须全部存在或全部不存在。如果存在，每个名称代表指定类型的一项（参数或结果），并且签名中的所有非[空白](https://go.dev/ref/spec#Blank_identifier)名称必须是[唯一的](https://go.dev/ref/spec#Uniqueness_of_identifiers)。如果不存在，则每种类型代表该类型的一项。

##### 2.9.2Result唯一可以无括号

Parameter and result lists are always parenthesized except that if there is exactly one unnamed result it may be written as an unparenthesized type.

##### 2.9.3       ...可变参数 ，附加：必须在参数列表最后面

The final incoming parameter in a function signature may have a type prefixed with `...`. A function with such a parameter is called *variadic* and may be invoked with zero or more arguments for that parameter.

函数签名中的最终传入参数可能具有以 为前缀的类型`...`。具有此类参数的函数称为*可变参数，*并且可以为该参数使用零个或多个参数调用。

```
func()
func(x int) int
func(a, _ int, z float32) bool
func(a, b int, z float32) (bool)
func(prefix string, values ...int)
func(a, b int, z float64, opt ...interface{}) (success bool)
func(int, int, float64) (float64, *[]int)
func(n int) func(p *T)
```

### 2.10Interface types

An interface type specifies a [method set](https://go.dev/ref/spec#Method_sets) called its *interface*. 一个指定了方法集的接口类型 称为interface

##### 疑问：斜体的interface？

A variable of interface type can store a value of any type with a method set that is any superset of the interface.

接口类型的变量可以存储任何类型的值，方法集是接口的任何**超集**。

 Such a type is said to *implement the interface*. 

##### 特点：未初始化的值为nil 

The value of an uninitialized variable of interface type is `nil`.

```
InterfaceType      = "interface" "{" { ( MethodSpec | InterfaceTypeName ) ";" } "}" .
MethodSpec         = MethodName Signature .
MethodName         = identifier .
InterfaceTypeName  = TypeName .
```

##### 接口方法2种方式

An interface type may specify methods *explicitly* through method specifications, or it may *embed* methods of other interfaces through interface type names.一个接口类型   要么显式的声明方法，要么 通过接口类型名    内嵌其他接口的方法

```
// A simple File interface.
interface {
	Read([]byte) (int, error)
	Write([]byte) (int, error)
	Close() error
}
```

###### 显式

The name of each explicitly specified method must be [unique](https://go.dev/ref/spec#Uniqueness_of_identifiers) and not [blank](https://go.dev/ref/spec#Blank_identifier).显式指定的方法必须唯一 不为Blank

```
interface {
	String() string
	String() string  // illegal: String not unique
	_(x int)         // illegal: method must have non-blank name
}
```

More than one type may implement an interface. For instance, if two types `S1` and `S2` have the method set

```
func (p T) Read(p []byte) (n int, err error)
func (p T) Write(p []byte) (n int, err error)
func (p T) Close() error
```

(where `T` stands for either `S1` or `S2`) then the `File` interface is implemented by both `S1` and `S2`, regardless of what other methods `S1` and `S2` may have or share.

A type implements any interface comprising any subset of its methods and may therefore implement several distinct interfaces. For instance, all types implement the *empty interface*:

```
interface{}
```

Similarly, consider this interface specification, which appears within a [type declaration](https://go.dev/ref/spec#Type_declarations) to define an interface called `Locker`:

```
type Locker interface {
	Lock()
	Unlock()
}
```

If `S1` and `S2` also implement

```
func (p T) Lock() { … }
func (p T) Unlock() { … }
```

they implement the `Locker` interface as well as the `File` interface.

An interface `T` may use a (possibly qualified) interface type name `E` in place of a method specification. This is called *embedding* interface `E` in `T`. The [method set](https://go.dev/ref/spec#Method_sets) of `T` is the *union* of the method sets of `T`’s explicitly declared methods and of `T`’s embedded interfaces.

```
type Reader interface {
	Read(p []byte) (n int, err error)
	Close() error
}

type Writer interface {
	Write(p []byte) (n int, err error)
	Close() error
}

// ReadWriter's methods are Read, Write, and Close.
type ReadWriter interface {
	Reader  // includes methods of Reader in ReadWriter's method set
	Writer  // includes methods of Writer in ReadWriter's method set
}
```

A *union* of method sets contains the (exported and non-exported) methods of each method set exactly once, and methods with the [same](https://go.dev/ref/spec#Uniqueness_of_identifiers) names must have [identical](https://go.dev/ref/spec#Type_identity) signatures.

```
type ReadCloser interface {
	Reader   // includes methods of Reader in ReadCloser's method set
	Close()  // illegal: signatures of Reader.Close and Close are different
}
```

An interface type `T` may not embed itself or any interface type that embeds `T`, recursively.

```
// illegal: Bad cannot embed itself
type Bad interface {
	Bad
}

// illegal: Bad1 cannot embed itself using Bad2
type Bad1 interface {
	Bad2
}
type Bad2 interface {
	Bad1
}
```

### 2.11Map types

A map is an unordered group of elements of one type, called the element type, indexed by a set of unique *keys* of another type, called the key type. The value of an uninitialized map is `nil`.

map是无序的，通过另一个type的key索引得到。未初始化的Map值为nil

```
MapType     = "map" "[" KeyType "]" ElementType .
KeyType     = Type .
```

#### 2.11.1key的可比较性

The [comparison operators](https://go.dev/ref/spec#Comparison_operators) `==` and `!=` must be fully defined for operands of the key type; 比较运算符必须是 在key type 的操作数 已经被定义的。

也就是说key值必须可以比较。

thus the key type must not be a function, map, or slice. key的类型不可以是func,map,slice

If the key type is an interface type, these comparison operators must be defined for the dynamic key values; failure will cause a [run-time panic](https://go.dev/ref/spec#Run_time_panics).

如果是接口类型，比较运算符必须定义在他们的动态类型种。失败的话会造成run-time panic

```
map[string]int
map[*T]struct{ x, y float64 }
map[string]interface{}
```

#### 2.11.2map和slice一样运行时变长，可扩容

The number of map elements is called its length. For a map `m`, it can be discovered using the built-in function [`len`](https://go.dev/ref/spec#Length_and_capacity) and may change during execution. 

#### 2.11.3添加和删除map里的元素，

Elements may be added during execution using [assignments](https://go.dev/ref/spec#Assignments) and retrieved with [index expressions](https://go.dev/ref/spec#Index_expressions); they may be removed with the [`delete`](https://go.dev/ref/spec#Deletion_of_map_elements) built-in function.

添加通过  map [key] *=  value;   获得通过 map[key]；删除通过 deletion。详细的点开 对应连接十分明白。  删除的语法可能要到内置函数看具体的。另写一篇笔记

#### 2.11.4make内置函数创建Map

A new, empty map value is made using the built-in function [`make`](https://go.dev/ref/spec#Making_slices_maps_and_channels), which takes the map type and an optional capacity hint as arguments:

```
make(map[string]int)
make(map[string]int, 100)
```

##### 2.11.5 空map和Nil map不同

The initial capacity does not bound its size: maps grow to accommodate the number of items stored in them, with the exception of `nil` maps. A `nil` map is equivalent to an empty map except that no elements may be added.

初始容量不限制其大小：map可以增长以容纳存储在其中的数据条目，`nil`map相当于`empty` map,但是nil map不可以添加没有元素。

### 2.12Channel types:无名之辈

A channel provides a mechanism for [concurrently executing functions](https://go.dev/ref/spec#Go_statements) to communicate by [sending](https://go.dev/ref/spec#Send_statements) and [receiving](https://go.dev/ref/spec#Receive_operator) values of a specified element type.

通道为[并发执行函数](https://go.dev/ref/spec#Go_statements)提供了一种机制， 通过[发送](https://go.dev/ref/spec#Send_statements)和 [接收](https://go.dev/ref/spec#Receive_operator) 指定元素类型的值来进行通信 。未初始化通道的值为`nil`。

 The value of an uninitialized channel is `nil`.不初始化就是zero value  值为nil

```
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
```

#### 收还是发？

The optional `<-` operator specifies the channel *direction*, *send* or *receive*.

#### 默认双向

If no direction is given, the channel is *bidirectional*. A channel may be constrained only to send or only to receive by [assignment](https://go.dev/ref/spec#Assignments) or explicit [conversion](https://go.dev/ref/spec#Conversions).

```
chan T          // can be used to send and receive values of type T
chan<- float64  // can only be used to send float64s       chan 是 管道  进管道 就是发送
<-chan int      // can only be used to receive ints                     出管道 就是接收

```

The `<-` operator associates with the leftmost `chan` possible:

```
chan<- chan int    // same as chan<- (chan int)
chan<- <-chan int  // same as chan<- (<-chan int)
<-chan <-chan int  // same as <-chan (<-chan int)
chan (<-chan int)
```

#### make内置函数

A new, initialized channel value can be made using the built-in function [`make`](https://go.dev/ref/spec#Making_slices_maps_and_channels), which takes the channel type and an optional *capacity* as arguments:

```
make(chan int, 100)
```

##### 有疑问对于这个管道的buffer化

The capacity, in number of elements, sets the size of the buffer in the channel. 

##### 通信条件

If the capacity is zero or absent, the channel is unbuffered and communication succeeds only when both a sender and receiver are ready. Otherwise, the channel is buffered and communication succeeds without blocking if the buffer is not full (sends) or not empty (receives). 

如果cap是0或不存在，管道没有被Buffer ，通信成功只有 收发双发准备好才行。 如果管道是Buffer好的，通信成功当这个发送方Buffer 没满  接收方buffer 非空。

有东西可发 有空间可接收的意思吗？

##### nil管道不可用，未初始化

A `nil` channel is never ready for communication.

#### 关闭管道close

A channel may be closed with the built-in function [`close`](https://go.dev/ref/spec#Close). The multi-valued assignment form of the [receive operator](https://go.dev/ref/spec#Receive_operator) reports whether a received value was sent before the channel was closed.

##### 疑问

A single channel may be used in [send statements](https://go.dev/ref/spec#Send_statements), [receive operations](https://go.dev/ref/spec#Receive_operator), and calls to the built-in functions [`cap`](https://go.dev/ref/spec#Length_and_capacity) and [`len`](https://go.dev/ref/spec#Length_and_capacity) by any number of goroutines without further synchronization.

Channels act as first-in-first-out queues.管道就是先进先出队列 

 For example, if one goroutine sends values on a channel and a second goroutine receives them, the values are received in the order sent.

例如，如果一个 goroutine 在通道上发送值，而第二个 goroutine 接收它们，则按照发送的顺序接收值。

