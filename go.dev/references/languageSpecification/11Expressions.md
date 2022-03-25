[TOC]



# Expressions表达式

An expression specifies the computation of a value by applying operators and functions to operands.

表达式通过对操作数应用运算符和函数来指定值的计算。

## 1Operands操作数

Operands denote the elementary values in an expression. An operand may be a literal, a (possibly [qualified](https://go.dev/ref/spec#Qualified_identifiers)) non-[blank](https://go.dev/ref/spec#Blank_identifier) identifier denoting a [constant](https://go.dev/ref/spec#Constant_declarations), [variable](https://go.dev/ref/spec#Variable_declarations), or [function](https://go.dev/ref/spec#Function_declarations), or a parenthesized expression.

操作数表示表达式中的基本值。操作数可以是字面值，表示常量、变量或函数的(可能限定的)非空标识符，或括号表达式。

The [blank identifier](https://go.dev/ref/spec#Blank_identifier) may appear as an operand only on the left-hand side of an [assignment](https://go.dev/ref/spec#Assignments).

空白标识符只能作为操作数出现在赋值的左侧。

```
Operand     = Literal | OperandName | "(" Expression ")" .
Literal     = BasicLit | CompositeLit | FunctionLit .
BasicLit    = int_lit | float_lit | imaginary_lit | rune_lit | string_lit .
OperandName = identifier | QualifiedIdent .
```

## 2Qualified identifiers限定标识符

A qualified identifier is an identifier qualified with a package name prefix. Both the package name and the identifier must not be [blank](https://go.dev/ref/spec#Blank_identifier).

限定标识符是具有包名称前缀的限定标识符。包名和标识符都不能为空。

```
QualifiedIdent = PackageName "." identifier .
```

A qualified identifier accesses an identifier in a different package, which must be [imported](https://go.dev/ref/spec#Import_declarations). The identifier must be [exported](https://go.dev/ref/spec#Exported_identifiers) and declared in the [package block](https://go.dev/ref/spec#Blocks) of that package.

限定标识符访问不同包中的标识符，`package`必须被导入。标识符必须是`exported identifier`并在该包的包块中声明。

```
math.Sin	// denotes the Sin function in package math
```

## 3Composite literals复合字面量

[^]: 这部分完全可以放入4Lexicial element

### 3.1分别有 struct,array,slice,map,a new value 五种复合CLit

Composite literals construct values for `structs`, `arrays`, `slice`s, and `maps` and create a new value each time they are evaluated. They consist of the type of the literal followed by a brace-bound list of elements. Each element may optionally be preceded by a corresponding key.

复合字面量 构造结构、数组、片和映射的值，并在每次计算它们时创建一个新值。它们由 字面量类型 后面跟着 大括号括起来的元素列表  构成。每个元素前面可以有一个对应的键。

```
CompositeLit  = LiteralType LiteralValue .
LiteralType   = StructType | ArrayType | "[" "..." "]" ElementType |
                SliceType | MapType | TypeName .     这里的Typename应该可以把管道类型包括进来。
LiteralValue  = "{" [ ElementList [ "," ] ] "}" .
                      ElementList   = KeyedElement { "," KeyedElement } .

KeyedElement  = [ Key ":" ] Element .
                  Key           = FieldName | Expression | LiteralValue .
                                  FieldName     = identifier .
                            Element       = Expression | LiteralValue .
```

The LiteralType's underlying type must be a struct, array, slice, or map type (the grammar enforces this constraint except when the type is given as a TypeName).LiteralType 的基础类型必须是 struct、 array、 slice 或 map 类型(语法强制执行此约束，除非该类型作为 TypeName 给出)。 

#### value的assignability

The types of the elements and keys must be [assignable](https://go.dev/ref/spec#Assignability) to the respective field, element, and key types of the literal type; there is no additional conversion.元素和键的类型必须可以分配给文本类型的各个字段、元素和键类型; 不存在额外的转换。 

### 3.2key的含义

The key is interpreted as a field name for struct literals, an index for array and slice literals, and a key for map literals. For `map` literals, all elements must have a key. 键被解释为结构文本的字段名、数组和片文本的索引以及映射文本的键。对于`map`字面量，所有元素都必须有一个键.

所以通过Key的存在检验element的存在

It is an error to specify multiple elements with the same field name or constant key value. For non-constant map keys, see the section on [evaluation order](https://go.dev/ref/spec#Order_of_evaluation).指定具有相同字段名或常量键值的多个元素是错误的。有关非常量映射键，请参阅计算顺序一节。

For struct literals the following rules apply:对于结构文本，应用以下规则:



- A key must be a field name declared in the struct type.

  键必须是在结构类型中声明的字段名。

- An element list that does not contain any keys must list an element for each struct field in the order in which the fields are declared.

  不包含任何键的元素列表必须按声明字段的顺序列出每个结构字段的元素。

- If any element has a key, every element must have a key.如果任何元素都有键，那么每个元素都必须有键。

  #### 字段可以有键但不一定需要显式指明element

- An` Element List` that contains keys does not need to have an element for each struct field. `Omitted fields` get the zero value for that field.

  包含键的元素列表不需要为每个结构字段包含元素。被省略的字段将获得该字段的零值。

- A literal may omit the element list; such a literal evaluates to the zero value for its type.

  字面量可以省略元素列表; 这样的常值的计算结果为其类型的零值。比如 []int {}  每个值都是0

- It is an error to specify an element for a non-exported field of a struct belonging to a different package.

  为属于不同包的结构的非导出字段 指定元素   是错误的。

Given the declarations

考虑到这些声明

```
type Point3D struct { x, y, z float64 }
type Line struct { p, q Point3D }
```

one may write

你可以写

```
origin := Point3D{}                            // zero value for Point3D
line := Line{origin, Point3D{y: -4, z: 12.3}}  // zero value for line.q.x
```

### 3.3数组和切片

For array and slice literals the following rules apply:对于数组和片字面量，应用以下规则:

- Each element has an associated integer index marking its position in the array.

  每个元素都有一个关联的整数索引，标记它在数组中的位置。

- An element with a key uses the key as its index. The key must be a non-negative constant [representable](https://go.dev/ref/spec#Representability) by a value of type `int`; and if it is typed it must be of integer type.

  如果具有键的元素使用键作为其索引。键必须是一个非负常数，可由 int 类型的值表示; 如果键值是类型化的，则必须是整数类型的。

- An element without a key uses the previous element's index plus one. If the first element has no key, its index is zero.

  没有键的元素使用前一个元素的索引加1。如果第一个元素没有键，则其索引为零。

[Taking the address](https://go.dev/ref/spec#Address_operators) of a composite literal generates a pointer to a unique [variable](https://go.dev/ref/spec#Variables) initialized with the literal's value.

获取复合字面量 的地址会生成一个指向用该字面量的值初始化的唯一变量的指针。

```
var pointer *Point3D = &Point3D{y: 1000}
```

##### map|slice:初始化 和empty是不同的。zero value是nil 而 初始化的是empty不为nil

Note that the [zero value](https://go.dev/ref/spec#The_zero_value) for a slice or map type is not the same as an initialized but empty value of the same type. Consequently, taking the address of an empty slice or map composite literal does not have the same effect as allocating a new slice or map value with [new](https://go.dev/ref/spec#Allocation).

请注意，片或映射类型的零值与同一类型的已初始化但为空的值不同。因此，获取空片或映射复合文字的地址与使用 new 分配新片或映射值的效果不同。

```
p1 := &[]int{}    // p1 points to an initialized, empty slice with value []int{} and length 0
p2 := new([]int)  // p2 points to an uninitialized slice with value nil and length 0
```



##### 数组长度和索引

The length of an array literal is the length specified in the literal type. If fewer elements than the length are provided in the literal, the missing elements are set to the zero value for the array element type. It is an error to provide elements with index values outside the index range of the array. The notation `...` specifies an array length equal to the maximum element index plus one.

数组字面量的长度 是 在字面量类型 部分中的指定的长度部分。【数组类型是包含 长度这一信息的，参考7GoTypes.md】。如果字面量中提供的元素少于长度，则缺少的元素将设置为数组元素类型的零值。提供索引值超出数组索引范围的元素是错误的。表示法... 指定的数组长度等于最大元素索引加1。

```
buffer := [10]string{}             // len(buffer) == 10
intSet := [6]int{1, 2, 3, 5}       // len(intSet) == 6
days := [...]string{"Sat", "Sun"}  // len(days) == 2
```

A slice literal describes the entire underlying array literal. Thus the length and capacity of a slice literal are the maximum element index plus one. A slice literal has the form

切片字面量描述整个底层的数组字面量。因此，片切片字面量的长度和容量是最大元素索引加1。一个 slice literal 具有这样的形式

```
[]T{x1, x2, … xn}
```

and is shorthand for a slice operation applied to an array:

它是应用于数组的 slice 操作的简写形式:

```
tmp := [n]T{x1, x2, … xn}
tmp[0 : n]
```



### 3.4array,slice,map复合字面量的省略表达

#### 普通类型的省略：有点问题 数组类型应该是包含长度的。

Within a composite literal of array, slice, or map type `T`, elements or map keys that are themselves composite literals

​     may elide the respective literal type if it is identical to the element or key type of `T`. 

#### 取地址符的省略

Similarly, elements or keys that are addresses of composite literals

​    may elide the `&T` when the element or key type is `*T`.

在由数组、片或映射类型 `T` 组成的复合文字中，如果元素或映射键本身是复合字面量，那么如果它们与元素或键类型 `T` 相同，那么这些元素或键可能会省略相应的字面量类型。

同样的，元素或键是 复合字面量的地址，如果 元素或键的类型 是`*T`那么 元素或键可能省略`&T`

```
复合字面量                       元素类型          元素类型是复合类型
```

```
[...]Point{{1.5, -3.5}, {0, 0}}  Point                 2维   
// same as [...]Point{Point{1.5, -3.5}, Point{0, 0}} 
[][]int{{1, 2, 3}, {4, 5}}       []int                 一维数组
// same as [][]int{[]int{1, 2, 3}, []int{4, 5}}
[][]Point{{{0, 1}, {1, 2}}}      []Point               一维数组
// same as [][]Point{[]Point{Point{0, 1}, Point{1, 2}}}

map[string]Point{"orig": {0, 0}}  Point{}               是      这里的复合字面量是 Point
// same as map[string]Point{"orig": Point{0, 0}}
```



```
复合字面量                             键                键是复合类型
map[Point]string{{0, 0}: "orig"}    Point{}               是
// same as map[Point]string{Point{0, 0}: "orig"}                    
```

```
地址

type PPoint *Point
[2]*Point{{1.5, -3.5}, {}}    // same as [2]*Point{&Point{1.5, -3.5}, &Point{}}  省略了&T
[2]PPoint{{1.5, -3.5}, {}}    // same as [2]PPoint{PPoint(&Point{1.5, -3.5}), PPoint(&Point{})} 省略了 T
```

### 3.5不省略表达时，使用（）解决{}带来的歧义

A parsing ambiguity arises when a composite literal using the TypeName form of the LiteralType appears as an operand between the [keyword](https://go.dev/ref/spec#Keywords) and the opening brace of the block of an "if", "for", or "switch" statement, and the composite literal is not enclosed in `parentheses`(), `square brackets[]`, or `curly braces{}`. In this rare case, the opening brace of the literal is erroneously parsed as the one introducing the block of statements. To resolve the ambiguity, the composite literal must appear within `parentheses()`.

当使用 LiteralType  的TypeName 形式`也就是不省略表达`的复合文本作为操作数出现在“ if”、“ for”或“ switch”语句的关键字和块的开大括号之间，并且复合文本不包含在括号()、方括号[]或花括号{}中时，就会出现解析歧义。在这种罕见的情况下，文字的开括号被错误地解析为引入语句块的括号。若要解决歧义，复合文本必须出现在括号中。

```
if x == (T{a,b,c}[i]) { … }  用parentheses() 括号括起来
if (x == T{a,b,c}[i]) { … }
```

Examples of valid array, slice, and map literals:

有效数组、片段和映射文字的示例:

```
// list of prime numbers
primes := []int{2, 3, 5, 7, 9, 2147483647}

// vowels[ch] is true if ch is a vowel
vowels := [128]bool{'a': true, 'e': true, 'i': true, 'o': true, 'u': true, 'y': true}

// the array [10]float32{-1, 0, 0, 0, -0.1, -0.1, 0, 0, 0, -1}
filter := [10]float32{-1, 4: -0.1, -0.1, 9: -1}

// frequencies in Hz for equal-tempered scale (A4 = 440Hz)
noteFrequency := map[string]float32{
	"C0": 16.35, "D0": 18.35, "E0": 20.60, "F0": 21.83,
	"G0": 24.50, "A0": 27.50, "B0": 30.87,
}
```

## 4Function literals函数文字

### 匿名函数定义：缺少FuncName

A function literal represents an anonymous [function](https://go.dev/ref/spec#Function_declarations).函数字面量表示一个匿名函数。

```
FunctionLit = "func" Signature FunctionBody .
func(a, b int, z float64) bool { return a*b < int(z) }
```

A function literal can be assigned to a variable or invoked directly.可以为变量赋值或直接调用匿名函数。

### 匿名函数的直接调用（）

```
f := func(x, y int) int { return x + y }赋值给变量
func(ch chan int) { ch <- ACK }(replyChan) 字面量直接调用 （）里面时传入的参数
```

### 疑问  surrounding function

参考pkg global下面的

```
func runInitiator(first, last int, app *iris.Application) {
   wg := sync.WaitGroup{}
   wg.Add(last - first + 1)
   for j := first; j < last+1; j++ {
      go func(j int) {
         if initiators[j].Level >= 10 {
            mu.Lock()
         }
         initiators[j].Action(app)
         if initiators[j].Level >= 10 {
            mu.Unlock()
         }
         wg.Done()
      }(j)
   }
   wg.Wait()
}

```

```
这个是匿名函数，需要参数j
func(j int) {
         if initiators[j].Level >= 10 {
            mu.Lock()
         }
         initiators[j].Action(app)
         if initiators[j].Level >= 10 {
            mu.Unlock()
         }
         wg.Done()
      }
 j的初始值 是runInitiator函数形参first的实参value
 这里来说  匿名函数 引用了surrounding function也就是这里的runInitiator函数 的变量first;first变量 是在匿名函数和runInitiator之间共享的，只要变量first可以被访问到，匿名函数和runInit继续存在
```

Function literals are *closures*: they may refer to variables defined in a surrounding function. Those variables are then shared between the surrounding function and the function literal, and they survive as long as they are accessible.

函数字面量是闭包: 它们可以引用在周围函数中定义的变量。然后，这些变量在周围的函数和匿名函数之间共享，只要它们可以被访问，它们就会继续存在。

## 5Primary expressions主要表达式

Primary expressions are the operands for unary and binary expressions.

主表达式是一元和二元表达式的操作数。

```
PrimaryExpr =
	Operand |               操作数
	Conversion |            转变
	MethodExpr |            方法表达式
	PrimaryExpr Selector |  选择器
	PrimaryExpr Index |
	PrimaryExpr Slice |
	PrimaryExpr TypeAssertion |
	PrimaryExpr Arguments .

Selector       = "." identifier .
Index          = "[" Expression "]" .
Slice          = "[" [ Expression ] ":" [ Expression ] "]" |
                 "[" [ Expression ] ":" Expression ":" Expression "]" .  //????
TypeAssertion  = "." "(" Type ")" .
Arguments      = "(" [ ( ExpressionList | Type [ "," ExpressionList ] ) [ "..." ] [ "," ] ] ")" .
x
2
(s + ".txt")
f(3.1415, true)
Point{1, 2}
m["foo"]
s[i : j + 1]
obj.color
f.p[i].x()
```

## 6Selectors选择器:区别于限定标识符QI

For a [primary expression](https://go.dev/ref/spec#Primary_expressions) `x` that is not a [package name](https://go.dev/ref/spec#Package_clause), the *selector expression*

对于不是包名称的主表达式 x，选择器表达式

```
x.f
```

denotes the field or method `f` of the value `x` (or sometimes `*x`; see below). 

### selector定义

The identifier `f` is called the (field or method) *selector*; it must not be the [blank identifier](https://go.dev/ref/spec#Blank_identifier). The type of the selector expression is the type of `f`. If `x` is a package name, see the section on [qualified identifiers](https://go.dev/ref/spec#Qualified_identifiers).

表示值 x 的字段或方法 f (有时是 * x; 见下文)。标识符 f 称为(字段或方法)选择器; 它不能是空标识符。选择器表达式的类型是 f 类型。如果 x 是一个包名，请参阅限定标识符部分。

#### 深度depth定义

#### 注意：不需要 x.embededField.f  x.f是可行的

A selector `f` may denote a field or method `f` of a type `T`, or it may refer to a field or method `f` of a nested [embedded field](https://go.dev/ref/spec#Struct_types) of `T`. The number of embedded fields traversed to reach `f` is called its *depth* in `T`. The depth of a field or method `f` declared in `T` is zero. The depth of a field or method `f` declared in an embedded field `A` in `T` is the depth of `f` in `A` plus one.

选择器 f 可以表示类型 t 的字段或方法 f，也可以表示嵌套嵌入字段 t 的字段或方法 f。穿越到达 f 的嵌入域数称为它在 `T` 中的深度。在 `T` 中声明的字段或方法 f 的深度为零。在嵌入字段 a 中声明的字段或方法 f 的深度是 a 加1中 f 的深度。

#### 一些约定

The following rules apply to selectors:以下规则适用于选择器:

#####      非指针非接口要求 有一个最简单shallowest的slector

1. For a value `x` of type `T` or `*T` where `T` is not a pointer or interface type, `x.f` denotes the field or method at the shallowest depth in `T` where there is such an `f`. If there is not exactly [one `f`](https://go.dev/ref/spec#Uniqueness_of_identifiers) with shallowest depth, the selector expression is illegal.

   对于 t 或 * t 类型的值 x，其中 `T`不是指针或接口类型【之前就提过的要求】，`x.f `表示 t 中最浅深度处的字段或方法，存在 f。如果没有一个具有最浅深度的 f，则选择器表达式是非法的。

   ##### 接口的动态调用

2. For a value `x` of type `I` where `I` is an interface type, `x.f` denotes the actual method with name `f` of the dynamic value of `x`. If there is no method with name `f` in the [method set](https://go.dev/ref/spec#Method_sets) of `I`, the selector expression is illegal.

   对于 i 类型的值 x (其中 i 是接口类型) ，x.f 用动态值 x 的名称为 f 的实际的方法。如果在 i 的方法集中没有名为 f 的方法，则选择器表达式是非法的。

   ##### 指针选择器：只能field不能是method

3. As an exception, if the type of `x` is a [defined](https://go.dev/ref/spec#Type_definitions) pointer type and `(*x).f` is a valid selector expression denoting a field (but not a method), `x.f` is shorthand for `(*x).f`.

   作为一个例外，如果 x 的类型是定义的指针类型和`(* x).f `是表示字段(但不是方法)的有效选择器表达式，`x.f` 是`(* x).f`简写形式。

4. In all other cases, `x.f` is illegal.

   在所有其他情况下，x.f 都是非法的。

   ##### panic 发生在未初始化的情况下

5. If `x` is of pointer type and has the value `nil` and `x.f` denotes a struct field, assigning to or evaluating `x.f` causes a [run-time panic](https://go.dev/ref/spec#Run_time_panics).

   如果 x 是指针类型并且值为 nil，而 x.f 表示一个 struct 字段，那么赋值或计算 x.f 会导致运行时恐慌。nil代表未初始化所以panic

6. If `x` is of interface type and has the value `nil`, [calling](https://go.dev/ref/spec#Calls) or [evaluating](https://go.dev/ref/spec#Method_values) the method `x.f` causes a [run-time panic](https://go.dev/ref/spec#Run_time_panics).

   如果 x 是接口类型并且值为 nil，则调用或计算方法 x.f 将导致运行时恐慌。

For example, given the declarations:

例如，给定声明:

```
type T0 struct {
	x int
}

func (*T0) M0()
```

```
type T1 struct {
	y int
}

func (T1) M1()
```

```
type T2 struct {
	z int
	T1
	*T0
}

func (*T2) M2()
```

```
type Q *T2

var t T2     // with t.T0 != nil
var p *T2    // with p != nil and (*p).T0 != nil
var q Q = p
```

one may write:有人可能会写道:

```
t.z          // t.z
t.y          // t.T1.y
t.x          // (*t.T0).x

p.z          // (*p).z
p.y          // (*p).T1.y
p.x          // (*(*p).T0).x

q.x          // (*(*q).T0).x        (*q).x is a valid field selector

p.M0()       // ((*p).T0).M0()      M0 expects *T0 receiver
p.M1()       // ((*p).T1).M1()      M1 expects T1 receiver
p.M2()       // p.M2()              M2 expects *T2 receiver
t.M2()       // (&t).M2()           M2 expects *T2 receiver, see section on Calls  T不能调用*T的方法
```

but the following is invalid:但下面的内容是无效的:

```
q.M0()       // (*q).M0 is valid but not a field selector
```

## 7Method expressions方法表达式

If `M` is in the [method set](https://go.dev/ref/spec#Method_sets) of type `T`, `T.M` is a function that is callable as a regular function with the same arguments as `M` prefixed by an additional argument that is the receiver of the method.

如果 `M` 在`T` 类型的方法集 中，T.M 是一个可调用的普通函数，其参数与 `M`的参数相同，前缀是一个附加的参数，即方法的接收者。

```
MethodExpr    = ReceiverType "." MethodName .
ReceiverType  = Type .
```

Consider a struct type `T` with two methods, `Mv`, whose receiver is of type `T`, and `Mp`, whose receiver is of type `*T`.

考虑一个具有两个方法的 结构体类型 t，Mv (其接收器为 `T` 类型)和 Mp (其接收器为 `*T`类型)。 值类型，指针类型

```
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
```

The expression这个表达式

```
T.Mv//可以认为是一个函数字面量
```

yields a function equivalent to `Mv` but with an explicit receiver as its first argument; it has signature

产生一个等价于 Mv 的函数，但第一个参数是显式接收器; 它有签名

```
有点像转换
func(tv T, a int) int              闭包，直接传参调用
```

That function may be called normally with an explicit receiver, so these five invocations are equivalent:

这个函数可以通过一个显式`接收器`正常调用，所以这五个调用是等价的:

```
t.Mv(7)
T.Mv(t, 7)
(T).Mv(t, 7)
f1 := T.Mv; f1(t, 7) 函数字面量赋值给变量
f2 := (T).Mv; f2(t, 7)
```

Similarly, the expression

类似地，表达式

```
(*T).Mp
```

yields a function value representing `Mp` with signature

产生一个函数值，表示带有签名的 Mp

```
func(tp *T, f float32) float32
```

For a method with a value receiver, one can derive a function with an explicit pointer receiver, so

对于带有值接收器的方法，可以派生一个带有显式·`指针接收器`的函数，因此

```
(*T).Mv
```

yields a function value representing `Mv` with signature

产生一个函数值，表示带有签名的 Mv

```
func(tv *T, a int) int
```

Such a function indirects through the receiver to create a value to pass as the receiver to the underlying method; the method does not overwrite the value whose address is passed in the function call.

这样的函数通过接收器中转创建一个值，作为接收器传递给底层方法; 该方法不会覆盖地址在函数调用中传递的值。

The final case, a value-receiver function for a pointer-receiver method, is illegal because pointer-receiver methods are not in the method set of the value type.最后一种情况，即指针接收器方法的值接收器函数，是非法的，因为指针接收器方法不在值类型的方法集中。

##### 也就是 指针接收器范围更大

Function values derived from methods are called with function call syntax; the receiver is provided as the first argument to the call. That is, given `f := T.Mv`, `f` is invoked as `f(t, 7)` not `t.f(7)`. To construct a function that binds the receiver, use a [function literal](https://go.dev/ref/spec#Function_literals) or [method value](https://go.dev/ref/spec#Method_values).

从方法派生的函数值按照函数调用语法调用; 接收方作为调用的第一个参数提供。也就是说，给定 f: = T.Mv，f 被调用为 f (t，7)而不是 t.f (7)。要构造一个绑定接收方的函数，可以使用匿名函数或方法值。

It is legal to derive a function value from a method of an interface type. The resulting function takes an explicit receiver of that interface type.

从接口类型的方法派生函数值是合法的。结果函数接收该接口类型的显式接收器。

## 疑问：Method values方法值

If the expression `x` has static type `T` and `M` is in the [method set](https://go.dev/ref/spec#Method_sets) of type `T`, `x.M` is called a *method value*. The method value `x.M` is a function value that is callable with the same arguments as a method call of `x.M`. 如果表达式 x 具有静态类型 t，而 m 位于类型 t 的方法集中，则称 x.M 为方法值。方法值 x.M 是一个可调用的函数值，它具有与方法调用 x.M 相同的参数。

The expression `x` is evaluated and saved during the evaluation of the method value; the saved copy is then used as the receiver in any calls, which may be executed later.表达式 x 在方法值计算期间进行计算和保存; 然后在任何调用中将保存的副本用作接收方，这些调用可能在以后执行。

The type `T` may be an interface or non-interface type.类型 t 可以是接口类型或非接口类型。

As in the discussion of [method expressions](https://go.dev/ref/spec#Method_expressions) above, consider a struct type `T` with two methods, `Mv`, whose receiver is of type `T`, and `Mp`, whose receiver is of type `*T`.在上面对方法表达式的讨论中，考虑一个结构类型 t，它有两个方法，Mv (接收器是 t 型)和 Mp (接收器是 t 型)。

```
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
var pt *T
func makeT() T
```

The expression

```
t.Mv
```

yields a function value of type

生成类型的函数值

```
func(int) int
```

These two invocations are equivalent:

这两个调用是等价的:

```
t.Mv(7)
f := t.Mv; f(7)
```

Similarly, the expression

类似地，表达式

```
pt.Mp
```

yields a function value of type

生成类型的函数值

```
func(float32) float32
```

As with [selectors](https://go.dev/ref/spec#Selectors), a reference to a non-interface method with a value receiver using a pointer will automatically dereference that pointer: `pt.Mv` is equivalent to `(*pt).Mv`.

与选择器一样，使用指针对具有值接收器的非接口方法的引用将自动取消引用该指针: pt.Mv 等效于(* pt).Mv。

As with [method calls](https://go.dev/ref/spec#Calls), a reference to a non-interface method with a pointer receiver using an addressable value will automatically take the address of that value: `t.Mp` is equivalent to `(&t).Mp`.与方法调用一样，对具有使用可寻址值的指针接收器的非接口方法的引用将`自动获取该值的地址`: `t.Mp` 等效于`(& t).Mp`。

```
f := t.Mv; f(7)   // like t.Mv(7)
f := pt.Mp; f(7)  // like pt.Mp(7)
f := pt.Mv; f(7)  // like (*pt).Mv(7)
f := t.Mp; f(7)   // like (&t).Mp(7)
f := makeT().Mp   // invalid: result of makeT() is not addressable
```

Although the examples above use non-interface types, it is also legal to create a method value from a value of interface type.

尽管上面的示例使用非接口类型，但是从接口类型的值创建方法值也是合法的。

```
var i interface { M(int) } = myVal
f := i.M; f(7)  // like i.M(7)
```

## 8Index expressions索引表达式

A primary expression of the form主要表达式以下形式

```
a[x]
```

denotes the element of the array, pointer to array, slice, string or map `a` indexed by `x`. The value `x` is called the` *index* `or `*map key*`, respectively. The following rules apply:表示数组的元素、指向数组的指针、片、字符串或映射一个 x 索引的数组。值 x 分别称为索引键或映射键。以下规则适用:

If `a` is not a map:

- the index `x` must be of integer type or an untyped constant索引 x 必须是 integer 类型或非`untyped`常量
- a constant index must be non-negative and [representable](https://go.dev/ref/spec#Representability) by a value of type `int`常量索引必须是非负的，并且可以由 int 类型的值表示
- a constant index that is untyped is given type `int`  `untyped`的常量索引给定 int 类型
- the index `x` is *in range* if `0 <= x < len(a)`, otherwise it is *out of range*如果0 < = x < len (a) ，则 x 指数在范围内，否则它超出范围

#### 非map类型

For `a` of [array type](https://go.dev/ref/spec#Array_types) `A`:

对于 a 类型的数组:

- a [constant](https://go.dev/ref/spec#Constants) index must be in range

  常数索引必须在范围内
- if `x` is out of range at run time, a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs

  如果 x 在运行时超出范围，就会发生运行时恐慌
- `a[x]` is the array element at index `x` and the type of `a[x]` is the element type of `A`

  A [ x ]是索引 x 处的数组元素，a [ x ]的类型是 a 的元素类型

For `a` of [pointer](https://go.dev/ref/spec#Pointer_types) to array type:

对于指向数组类型的指针:

- `a[x]` is shorthand for `(*a)[x]`

  A [ x ]是(* a)[ x ]的简写

For `a` of [slice type](https://go.dev/ref/spec#Slice_types) `S`:对于 s 类型的片:

- if `x` is out of range at run time, a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs

  如果 x 在运行时超出范围，就会发生运行时恐慌
- `a[x]` is the slice element at index `x` and the type of `a[x]` is the element type of `S`

  A [ x ]是位于索引 x 处的 slice 元素，a [ x ]的类型是 s 的元素类型

For `a` of [string type](https://go.dev/ref/spec#String_types):

对于字符串类型:

- a [constant](https://go.dev/ref/spec#Constants) index must be in range if the string `a` is also constant如果字符串 a 也是常量，则常量索引必须在范围内
- if `x` is out of range at run time, a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs如果 x 在运行时超出范围，就会发生运行时恐慌
- `a[x]` is the non-constant byte value at index `x` and the type of `a[x]` is `byte`A [ x ]是索引 x 处的非常量字节值，而 a [ x ]的类型是字节
- `a[x]` may not be assigned to           A [ x ]不能被赋值，字符串一旦创建不可改变，`constant`

### map类型

For `a` of [map type](https://go.dev/ref/spec#Map_types) `M`:

对于类型为 m 的 a:

- `x`'s type must be [assignable](https://go.dev/ref/spec#Assignability) to the key type of `M`

  X 的类型必须可以分配给 m 的key
- if the map contains an entry with key `x`, `a[x]` is the map element with key `x` and the type of `a[x]` is the element type of `M`

  如果 map 包含一个带有键 x 的条目，那么 a [ x ]是带有键 x 的 map 元素，而 a [ x ]的类型是元素类型 
- if the map is `nil` or does not contain such an entry, `a[x]` is the [zero value](https://go.dev/ref/spec#The_zero_value) for the element type of `M`

  如果映射为 nil 或不包含这样的条目，则a[x]是m的元素类型 的0值

Otherwise `a[x]` is illegal.

否则 a [ x ]是非法的。

An index expression on a map `a` of type `map[K]V` used in an [assignment](https://go.dev/ref/spec#Assignments) or initialization of the special form

s类型为map[k]V的map值 的 索引表表达式 是用来 赋值 或特殊形式的初始化

```
v, ok = a[x]
v, ok := a[x]
var v, ok = a[x]
```

yields an additional untyped boolean value. The value of `ok` is `true` if the key `x` is present in the map, and `false` otherwise.

会产生一个额外的无类型布尔值。如果键 x 出现在 map 中，则 ok 的值为 true，否则为 false。

Assigning to an element of a `nil` map causes a [run-time panic](https://go.dev/ref/spec#Run_time_panics).分配给 nil 映射的元素会导致运行时恐慌。需要初始化才能分配

## 9Slice expressions切片表达式

Slice expressions construct a substring or slice from a string, array, pointer to array, or slice. There are two variants: a simple form that specifies a low and high bound, and a full form that also specifies a bound on the capacity.

切片表达式从字符串、数组、指向数组的指针或片构造子字符串或片。有两种变体: 一种是指定上下限范围的简单形式，另一种是也指定容量界限的完整形式。

### Simple slice expressions简单切片表达式

#### 针对对象：字符串，数组，数组指针，切片

For a string, array, pointer to array, or slice `a`, the primary expression

对于字符串、数组、指向数组的指针或片 a，则为主表达式

```
a[low : high]
```

constructs a substring or slice. The *indices* `low` and `high` select which elements of operand `a` appear in the result. The result has indices starting at 0 and length equal to `high` - `low`. After slicing the array `a`

构造子字符串或片。索引的低和高选择哪些元素的操作数出现在结果中。结果是指数从0开始，长度等于高低。在对数组进行切片之后,

```
a := [5]int{1, 2, 3, 4, 5}
s := a[1:4]
```

the slice `s` has type `[]int`, length 3, capacity 4, and elements

 s 具有类型[] int、长度3、容量4和元素

```
s[0] == 2
s[1] == 3
s[2] == 4
```

For convenience, any of the indices may be omitted. A missing `low` index defaults to zero; a missing `high` index defaults to the length of the sliced operand:

为了方便起见，任何索引都可以省略。缺少的低索引缺省为零; 缺少的高索引缺省为分片操作数的长度:

```
a[2:]  // same as a[2 : len(a)]
a[:3]  // same as a[0 : 3]
a[:]   // same as a[0 : len(a)]
```

#### 指针索引`(*a)[low : high]`

If `a` is a pointer to an array, `a[low : high]` is shorthand for `(*a)[low : high]`.

如果 a 是指向数组的指针，则[ low: high ]表示(* a)[ low: high ]。

For arrays or strings, the indices are *in range* if `0` <= `low` <= `high` <= `len(a)`, otherwise they are *out of range*. For slices, the upper index bound is the slice capacity `cap(a)` rather than the length. A [constant](https://go.dev/ref/spec#Constants) index must be non-negative and [representable](https://go.dev/ref/spec#Representability) by a value of type `int`; for arrays or constant strings, constant indices must also be in range. If both indices are constant, they must satisfy `low <= high`. If the indices are out of range at run time, a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs.

对于数组或字符串，如果0 < = low < = high < = len (a) ，则索引在范围内，否则它们超出范围。对于切片，上索引界限是切片cap(a) ，而不是长度。常量索引必须是非负的，并且可由 int 类型的值表示; 对于数组或常量字符串，常量索引也必须在范围内。如果这两个指数都是常量，那么它们必须满足 low < = high。如果指数在运行时超出范围，就会发生运行时恐慌。

Except for [untyped strings](https://go.dev/ref/spec#Constants), if the sliced operand is a string or slice, the result of the slice operation is a non-constant value of the same type as the operand. For untyped string operands the result is a non-constant value of type `string`. If the sliced operand is an array, it must be [addressable](https://go.dev/ref/spec#Address_operators) and the result of the slice operation is a slice with the same element type as the array.

除了非类型化的字符串之外，如果被分片的操作数是字符串或片，片操作的结果是与操作数类型相同的非常量值。对于非类型字符串操作数，结果是类型为 string 的非常量值。如果切片操作数是一个数组，那么它必须是可寻址的，并且切片操作的结果是一个与数组具有相同元素类型的切片。

If the sliced operand of a valid slice expression is a `nil` slice, the result is a `nil` slice. Otherwise, if the result is a slice, it shares its underlying array with the operand.

如果有效片表达式的分片操作数为 nil 片，则结果为 nil 片。否则，如果结果是 slice，它将与操作数共享其基础数组。

```
var a [10]int
s1 := a[3:7]   // underlying array of s1 is array a; &s1[2] == &a[5]
s2 := s1[1:4]  // underlying array of s2 is underlying array of s1 which is array a; &s2[1] == &a[5]
s2[1] = 42     // s2[1] == s1[2] == a[5] == 42; they all refer to the same underlying array element
```

### Full slice expressions完整片段表达式

For an array, pointer to array, or slice `a` (but not a string), the primary expression

对于数组，指向数组的指针或切片(但不是字符串)是主表达式

```
a[low : high : max]
```

constructs a slice of the same type, and with the same length and elements as the simple slice expression `a[low : high]`. Additionally, it controls the resulting slice's capacity by setting it to `max - low`. Only the first index may be omitted; it defaults to 0. After slicing the array `a`

构造与简单片表达式 a [ low: high ]具有相同长度和元素的相同类型的片。此外，它通过将切片的容量设置为最大值-低来控制产生的切片的容量。只有第一个索引可以省略; 它的缺省值为0。在对数组进行切片之后,

```
a := [5]int{1, 2, 3, 4, 5}
t := a[1:3:5]
```

the slice `t` has type `[]int`, length 2, capacity 4, and elements

片 t 具有类型[] int、长度2、容量4和元素

```
t[0] == 2
t[1] == 3
```

As for simple slice expressions, if `a` is a pointer to an array, `a[low : high : max]` is shorthand for `(*a)[low : high : max]`. If the sliced operand is an array, it must be [addressable](https://go.dev/ref/spec#Address_operators).

对于简单的片表达式，如果 a 是指向数组的指针，则[ low: high: max ]是(* a)[ low: high: max ]的简写。如果被分片的操作数是数组，那么它必须是可寻址的。

The indices are *in range* if `0 <= low <= high <= max <= cap(a)`, otherwise they are *out of range*. A [constant](https://go.dev/ref/spec#Constants) index must be non-negative and [representable](https://go.dev/ref/spec#Representability) by a value of type `int`; for arrays, constant indices must also be in range. If multiple indices are constant, the constants that are present must be in range relative to each other. If the indices are out of range at run time, a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs.

如果0 < = 低 < = 高 < = max < = cap (a) ，则指数在范围内，否则超出范围。常量索引必须是非负的，并且可由 int 类型的值表示; 对于数组，常量索引也必须在范围内。如果多个索引是常量，则存在的常量必须在相对于彼此的范围内。如果指数在运行时超出范围，就会发生运行时恐慌。

## 10Type assertions类型断言



For an expression `x` of [interface type](https://go.dev/ref/spec#Interface_types) and a type `T`, the primary expression对于接口类型的表达式 x 和类型 t，主表达式

### x是接口类型interface{}【参考7GoTypes.md2.10】

```
x.(T)
```

asserts that `x` is not `nil` and that the value stored in `x`    is    of type `T`. The notation `x.(T)` is called a *type assertion*.

断言 x 不是nil，存储在 x 中的值是 t 类型的。记号 x (t)称为类型断言。

### 根据T是否是接口类型情况有二

#### T不是接口类型:x的动态类型与 实现x（接口类型）的T类型 类型一致

##### 动态类型 ，比如形参和实参的差别。接口的动态调用。接口变量由      实现该接口的类型的字面量 赋值

More precisely, if `T` is not an interface type, `x.(T)` asserts that the dynamic type of `x` is [identical](https://go.dev/ref/spec#Type_identity) to the type `T`. In this case, `T` must [implement](https://go.dev/ref/spec#Method_sets) the (interface) type of `x`; otherwise the type assertion is invalid since it is not possible for `x` to store a value of type `T`. 更准确地说，如果 `T`不是接口类型，则` x.(T)`断言 x 的动态类型与 `T` 类型相同。在这种情况下，`T`必须实现 x 的(接口)类型; 否则类型断言无效，因为 x 不可能存储类型 t 的值。

#### T是接口类型：x的动态类型 实现了 T接口。

If `T` is an interface type, `x.(T)` asserts that the dynamic type of `x` implements the interface `T`.如果 `T`是接口类型，则 `x.(T)`断言 x 的动态类型实现了接口 t。

If the type assertion holds, the value of the expression is the value stored in `x` and its type is `T`. If the type assertion is false, a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs. In other words, even though the dynamic type of `x` is known only at run time, the type of `x.(T)` is known to be `T` in a correct program.

如果类型断言成立，则表达式的值为存储在 x 中的值，其类型为 t。如果类型断言为 false，则会发生运行时恐慌。换句话说，即使只有在运行时才知道 x 的动态类型，但是在正确的程序中，x (t)的类型是 t。

```
var x interface{} = 7          // x has dynamic type int and value 7 空方法集的interface type
i := x.(int)                   // i has type int and value 7
根据 2Methods Set得知 任何类型都实现了空接口， 也就是int实现了 
type I interface { m() }

func f(y I) {
	s := y.(string)        // illegal: string does not implement I (missing method m)
	r := y.(io.Reader)     // r has type io.Reader and the dynamic type of y must implement both I and io.Reader
	…
}
```

### 类型断言的额外作用：是特殊形式的 赋值|初始化

A type assertion used in an [assignment](https://go.dev/ref/spec#Assignments) or initialization of the special form

在特殊窗体的赋值或初始化中使用的类型断言

```
v, ok = x.(T)  赋值
v, ok := x.(T) 初始化
var v, ok = x.(T)  f
var v, ok interface{} = x.(T) // dynamic types of v and ok are T and bool
```

yields an additional untyped boolean value. The value of `ok` is `true` if the assertion holds. Otherwise it is `false` and the value of `v` is the [zero value](https://go.dev/ref/spec#The_zero_value) for type `T`. No [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs in this case.

会产生一个额外的无类型布尔值。如果断言成立，则 ok 的值为 true。否则为 false，并且类型 t 的 v 值为零。在这种情况下，不会发生运行时恐慌。

## 11Calls调用

Given an expression `f` of function type `F`,给定函数类型 f 的表达式,

```
f(a1, a2, … an)
```

calls `f` with arguments `a1, a2, … an`. Except for one special case, arguments must be single-valued expressions [assignable](https://go.dev/ref/spec#Assignability) to the parameter types of `F` and are evaluated before the function is called. The type of the expression is the result type of `F`. A method invocation is similar but the method itself is specified as a selector upon a value of the receiver type for the method.

调用 f，参数 a1，a2，... an。除了一个特殊情况外，参数必须是单值表达式，可分配给 f 的参数类型，并在调用函数之前进行计算。表达式的类型是 f 的结果类型。方法调用是类似的，但是方法本身被指定为方法的接收方类型的值上的选择器。

```
math.Atan2(x, y)  // function call
var pt *Point
pt.Scale(3.5)     // method call with receiver pt
```

In a function call, the function value and arguments are evaluated in [the usual order](https://go.dev/ref/spec#Order_of_evaluation). After they are evaluated, the parameters of the call are passed by value to the function and the called function begins execution. The return parameters of the function are passed by value back to the caller when the function returns.在函数调用中，函数值和参数按照通常的顺序计算。计算之后，调用的参数通过值传递给函数，被调用的函数开始执行。当函数返回时，函数的返回参数通过值返回给调用方。

Calling a `nil` function value causes a [run-time panic](https://go.dev/ref/spec#Run_time_panics).调用 nil 函数值会导致运行时恐慌。

As a special case, if the return values of a function or method `g` are equal in number and individually assignable to the parameters of another function or method `f`, then the call `f(g(*parameters_of_g*))` will invoke `f` after binding the return values of `g` to the parameters of `f` in order. The call of `f` must contain no parameters other than the call of `g`, and `g` must have at least one return value. If `f` has a final `...` parameter, it is assigned the return values of `g` that remain after assignment of regular parameters.

作为一种特殊情况，如果一个函数或方法 g 的返回值在数量上是相等的，并且可以单独地分配给另一个函数或方法 f 的参数，那么调用 f (g (参数 _ of _ g))在依次将 g 的返回值绑定到 f 的参数之后将调用 f。F 的调用除了 g 的调用之外不能包含任何参数，g 必须至少有一个返回值。如果 f 有一个 final... 参数，它被赋值为 g 的返回值，这些返回值在赋值常规参数之后仍然存在。

```
func Split(s string, pos int) (string, string) {
	return s[0:pos], s[pos:]
}

func Join(s, t string) string {
	return s + t
}

if Join(Split(value, len(value)/2)) != value {
	log.Panic("test fails")
}
```

A method call `x.m()` is valid if the [method set](https://go.dev/ref/spec#Method_sets) of (the type of) `x` contains `m` and the argument list can be assigned to the parameter list of `m`. If `x` is [addressable](https://go.dev/ref/spec#Address_operators) and `&x`'s method set contains `m`, `x.m()` is shorthand for `(&x).m()`:

如果方法集(类型) x 包含 m，并且参数列表可以分配给参数列表 m，那么方法调用 x.m ()是有效的。如果 x 是可寻址的，而 & x 的方法集包含 m，那么 x.m ()是(& x)的简写形式。M () :

```
var p Point
p.Scale(3.5)
```

There is no distinct method type and there are no method literals.没有明确的方法类型，也没有方法文本。

## 12Passing arguments to `...` parameters将参数传递给... 参数

If `f` is [variadic](https://go.dev/ref/spec#Function_types) with a final parameter `p` of type `...T`, then within `f` the type of `p` is equivalent to type `[]T`. If `f` is invoked with no actual arguments for `p`, the value passed to `p` is `nil`. Otherwise, the value passed is a new slice of type `[]T` with a new underlying array whose successive elements are the actual arguments, which all must be [assignable](https://go.dev/ref/spec#Assignability) to `T`. The length and capacity of the slice is therefore the number of arguments bound to `p` and may differ for each call site.

如果f最后一个参数P是可变的，参数类型是类型`...T`，那么在函数f中p的类型等同于类型`[]T`。如果在没有实际参数的情况下调用 f，则传递给 p 的值为 nil。否则，传递的值是一个新的 类型[]T的切片，带有一个新的基础数组，其连续元素是实际的参数，所有这些参数都必须可以分配给 `T`。因此，切片的长度和容量是绑定到 p 的参数数量，对于每一次的调用[]T切片的长度和容量都可以不同。

### nil调用

Given the function and calls给定函数和调用

```
func Greeting(prefix string, who ...string)
Greeting("nobody")
Greeting("hello:", "Joe", "Anna", "Eileen")
```

within `Greeting`, `who` will have the value `nil` in the first call, and `[]string{"Joe", "Anna", "Eileen"}` in the second.

在 Greeting 中，第一个调用的值为 nil，第二个调用的值为[]字符串{“ Joe”、“ Anna”、“ Eileen”}。

### 特殊调用： variadic parameter...

If the final argument is assignable to a slice type `[]T` and is followed by `...`, it is passed unchanged as the value for a `...T` parameter. In this case no new slice is created.如果最后一个参数可以分配给一个 slice 类型[]T并且后面跟着... ，那么它将不变地作为... `T` 参数的值传递。在这种情况下，不会创建新切片。

Given the slice `s` and call给定切片和调用

```
s := []string{"James", "Jasmine"}
Greeting("goodbye:", s...)
```

within `Greeting`, `who` will have the same value as `s` with the same underlying array.

在 Greeting 中，它将具有与 s 相同的基础数组值。

## 13Operators

Operators combine operands into expressions.运算符将操作数组合成表达式。

```
Expression = UnaryExpr | Expression binary_op Expression .
UnaryExpr  = PrimaryExpr | unary_op UnaryExpr .

binary_op双目运算符  = "||" | "&&" | rel_op | add_op | mul_op .
rel_op  比   = "==" | "!=" | "<" | "<=" | ">" | ">=" .
add_op     = "+" | "-" | "|" | "^" .
mul_op     = "*" | "/" | "%" | "<<" | ">>" | "&" | "&^" .

unary_op   = "+" | "-" | "!" | "^" | "*" | "&" | "<-" .
```

Comparisons are discussed [elsewhere](https://go.dev/ref/spec#Comparison_operators). For other binary operators, the operand types must be [identical](https://go.dev/ref/spec#Type_identity) unless the operation involves shifts or untyped [constants](https://go.dev/ref/spec#Constants). For operations involving constants only, see the section on [constant expressions](https://go.dev/ref/spec#Constant_expressions).

比较在其他地方讨论。对于其他二进制运算符，操作数类型必须相同，除非该操作涉及移位或非类型化常量。对于仅涉及常量的操作，请参阅常量表达式一节。

Except for shift operations, if one operand is an untyped [constant](https://go.dev/ref/spec#Constants) and the other operand is not, the constant is implicitly [converted](https://go.dev/ref/spec#Conversions) to the type of the other operand.

除了移位操作之外，如果一个操作数是无类型常量，而另一个操作数不是，则该常量将隐式转换为另一个操作数的类型。

The right operand in a shift expression must have integer type or be an untyped constant [representable](https://go.dev/ref/spec#Representability) by a value of type `uint`. If the left operand of a non-constant shift expression is an untyped constant, it is first implicitly converted to the type it would assume if the shift expression were replaced by its left operand alone.

移位表达式中的右操作数必须具有整数类型，或者是可由 uint 类型的值表示的非类型常量。如果非常量移位表达式的左操作数是非类型化常量，则首先隐式转换为如果移位表达式被其左操作数单独替换时它将假定的类型。

```
var a [1024]byte
var s uint = 33

// The results of the following examples are given for 64-bit ints.
var i = 1<<s                   // 1 has type int
var j int32 = 1<<s             // 1 has type int32; j == 0
var k = uint64(1<<s)           // 1 has type uint64; k == 1<<33
var m int = 1.0<<s             // 1.0 has type int; m == 1<<33
var n = 1.0<<s == j            // 1.0 has type int; n == true
var o = 1<<s == 2<<s           // 1 and 2 have type int; o == false
var p = 1<<s == 1<<33          // 1 has type int; p == true
var u = 1.0<<s                 // illegal: 1.0 has type float64, cannot shift
var u1 = 1.0<<s != 0           // illegal: 1.0 has type float64, cannot shift
var u2 = 1<<s != 1.0           // illegal: 1 has type float64, cannot shift
var v float32 = 1<<s           // illegal: 1 has type float32, cannot shift
var w int64 = 1.0<<33          // 1.0<<33 is a constant shift expression; w == 1<<33
var x = a[1.0<<s]              // panics: 1.0 has type int, but 1<<33 overflows array bounds
var b = make([]byte, 1.0<<s)   // 1.0 has type int; len(b) == 1<<33

// The results of the following examples are given for 32-bit ints,
// which means the shifts will overflow.
var mm int = 1.0<<s            // 1.0 has type int; mm == 0
var oo = 1<<s == 2<<s          // 1 and 2 have type int; oo == true
var pp = 1<<s == 1<<33         // illegal: 1 has type int, but 1<<33 overflows int
var xx = a[1.0<<s]             // 1.0 has type int; xx == a[0]
var bb = make([]byte, 1.0<<s)  // 1.0 has type int; len(bb) == 0
```

### Operator precedence运算符优先级

Unary operators have the highest precedence. As the `++` and `--` operators form statements, not expressions, they fall outside the operator hierarchy. As a consequence, statement `*p++` is the same as `(*p)++`.

一元运算符的优先级最高。由于 + + 和 -- 运算符形成语句而不是表达式，因此它们不属于运算符层次结构。因此，语句 * p + + 与(* p) + + 相同。

There are five precedence levels for binary operators. Multiplication operators bind strongest, followed by addition operators, comparison operators, `&&` (logical AND), and finally `||` (logical OR):

二进制运算符有五个优先级。乘法运算符绑定最强，其次是加法运算符、比较运算符和 & (逻辑与) ，最后是 | | (逻辑 OR) :

```
Precedence    Operator
    5             *  /  %  <<  >>  &  &^
    4             +  -  |  ^
    3             ==  !=  <  <=  >  >=
    2             &&
    1             ||
```

Binary operators of the same precedence associate from left to right. For instance, `x / y * z` is the same as `(x / y) * z`.

相同优先级的二进制运算符从左到右关联。例如，x/y * z 与(x/y) * z 相同。

```
+x
23 + 3*x[i]
x <= f()
^a >> b
f() || g()
x == y+1 && <-chanInt > 0
```

### Arithmetic operators算术运算符

Arithmetic operators apply to numeric values and yield a result of the same type as the first operand. The four standard arithmetic operators (`+`, `-`, `*`, `/`) apply to integer, floating-point, and complex types; `+` also applies to strings. The bitwise logical and shift operators apply to integers only.

算术运算符应用于数值，并产生与第一个操作数类型相同的结果。四个标准算术运算符(+ ,-，* ,/)适用于整数、浮点和复杂类型; + 也适用于字符串。位逻辑运算符和移位运算符仅适用于整数。

```
+    sum                    integers, floats, complex values, strings
-    difference             integers, floats, complex values
*    product                integers, floats, complex values
/    quotient               integers, floats, complex values
%    remainder              integers

&    bitwise AND            integers
|    bitwise OR             integers
^    bitwise XOR            integers
&^   bit clear (AND NOT)    integers

<<   left shift             integer << integer >= 0
>>   right shift            integer >> integer >= 0
```

### Integer operators整数运算符

For two integer values `x` and `y`, the integer quotient `q = x / y` and remainder `r = x % y` satisfy the following relationships:

对于两个整数值 x 和 y，整数商 q = x/y 和余数 r = x% y 满足以下关系:

```
x = q*y + r  and  |r| < |y|
```

with `x / y` truncated towards zero (["truncated division"](https://en.wikipedia.org/wiki/Modulo_operation)).

X/y 缩短到零(“缩短除法”)。

```
 x     y     x / y     x % y
 5     3       1         2
-5     3      -1        -2
 5    -3      -1         2
-5    -3       1        -2
```

The one exception to this rule is that if the dividend `x` is the most negative value for the int type of `x`, the quotient `q = x / -1` is equal to `x` (and `r = 0`) due to two's-complement [integer overflow](https://go.dev/ref/spec#Integer_overflow):

这个规则的一个例外是，如果红利 x 是整数类型 x 的最负值，由于两个补整数溢出，商 q = x/-1等于 x (和 r = 0) :

```
			 x, q
int8                     -128
int16                  -32768
int32             -2147483648
int64    -9223372036854775808
```

If the divisor is a [constant](https://go.dev/ref/spec#Constants), it must not be zero. If the divisor is zero at run time, a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs. If the dividend is non-negative and the divisor is a constant power of 2, the division may be replaced by a right shift, and computing the remainder may be replaced by a bitwise AND operation:

如果除数是一个常数，它一定不能为零。如果在运行时除数为零，就会发生运行时恐慌。如果被除数是非负的，除数是2的常数幂，除法可以用右移代替，余数的计算可以用位与运算代替:

```
 x     x / 4     x % 4     x >> 2     x & 3
 11      2         3         2          3
-11     -2        -3        -3          1
```

The shift operators shift the left operand by the shift count specified by the right operand, which must be non-negative. If the shift count is negative at run time, a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs. The shift operators implement arithmetic shifts if the left operand is a signed integer and logical shifts if it is an unsigned integer. There is no upper limit on the shift count. Shifts behave as if the left operand is shifted `n` times by 1 for a shift count of `n`. As a result, `x << 1` is the same as `x*2` and `x >> 1` is the same as `x/2` but truncated towards negative infinity.

移位操作符通过右操作数指定的移位计数来移位左操作数，移位计数必须是非负的。如果在运行时移位计数为负，则会发生运行时恐慌。如果左操作数是有符号整数，则移位操作符实现算术移位; 如果是无符号整数，则实现逻辑移位。移位计数没有上限。对于 n 的移位计数，移位表现为左操作数移位 n 次1。因此，x < 1与 x * 2相同，x > 1与 x/2相同，只是向负无穷方向截断。

For integer operands, the unary operators `+`, `-`, and `^` are defined as follows:

对于整数操作数，一元运算符 + 、-和 ^ 的定义如下:

```
+x                          is 0 + x
-x    negation              is 0 - x
^x    bitwise complement    is m ^ x  with m = "all bits set to 1" for unsigned x
                                      and  m = -1 for signed x
```

### Integer overflow整数溢出

For unsigned integer values, the operations `+`, `-`, `*`, and `<<` are computed modulo 2*n*, where *n* is the bit width of the [unsigned integer](https://go.dev/ref/spec#Numeric_types)'s type. Loosely speaking, these unsigned integer operations discard high bits upon overflow, and programs may rely on "wrap around".

对于无符号整数值，+ 、-、 * 和 < < 都是计算模2n，其中 n 是无符号整数类型的位宽度。不严格地说，这些无符号整数操作在溢出时丢弃高位，而程序可能依赖于“换行”。

For signed integers, the operations `+`, `-`, `*`, `/`, and `<<` may legally overflow and the resulting value exists and is deterministically defined by the signed integer representation, the operation, and its operands. Overflow does not cause a [run-time panic](https://go.dev/ref/spec#Run_time_panics). A compiler may not optimize code under the assumption that overflow does not occur. For instance, it may not assume that `x < x + 1` is always true.

对于有符号整数，+ 、-、 * 、/和 < < 操作可以合法地溢出，结果值存在，并由有符号整数表示形式、操作数及其操作数确定。溢出不会引起运行时恐慌。在不发生溢出的假设下，编译器可能不会优化代码。例如，它可能不会假定 x < x + 1总是正确的。

### Floating-point operators浮点运算符

For floating-point and complex numbers, `+x` is the same as `x`, while `-x` is the negation of `x`. The result of a floating-point or complex division by zero is not specified beyond the IEEE-754 standard; whether a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs is implementation-specific.

对于浮点数和复数，+ x 等于 x，而-x 等于 x 的负数。在 ieee-754标准之外，没有指定浮点或复杂除零的结果; 是否发生运行时恐慌是特定于实现的。

An implementation may combine multiple floating-point operations into a single fused operation, possibly across statements, and produce a result that differs from the value obtained by executing and rounding the instructions individually. An explicit floating-point type [conversion](https://go.dev/ref/spec#Conversions) rounds to the precision of the target type, preventing fusion that would discard that rounding.

一个实现可以将多个浮点操作组合成一个单独的融合操作(可能跨语句) ，并产生与单独执行和舍入指令所获得的值不同的结果。显式的浮点类型转换回合到目标类型的精度，防止融合丢弃舍入。

For instance, some architectures provide a "fused multiply and add" (FMA) instruction that computes `x*y + z` without rounding the intermediate result `x*y`. These examples show when a Go implementation can use that instruction:

例如，一些体系结构提供了“融合乘法和加法”(FMA)指令，该指令计算 x * y + z，而不取整中间结果 x * y。这些例子显示了 Go 实现何时可以使用该指令:

```
// FMA allowed for computing r, because x*y is not explicitly rounded:
r  = x*y + z
r  = z;   r += x*y
t  = x*y; r = t + z
*p = x*y; r = *p + z
r  = x*y + float64(z)

// FMA disallowed for computing r, because it would omit rounding of x*y:
r  = float64(x*y) + z
r  = z; r += float64(x*y)
t  = float64(x*y); r = t + z
```

### String concatenation字符串串联

Strings can be concatenated using the `+` operator or the `+=` assignment operator:

字符串可以使用 + 运算符或 + = 赋值运算符串联:

```
s := "hi" + string(c)
s += " and good bye"
```

String addition creates a new string by concatenating the operands.

字符串加法通过串联操作数创建一个新字符串。

### Comparison operators比较操作符

Comparison operators compare two operands and yield an untyped boolean value.比较运算符比较两个操作数并产生一个无类型的布尔值。

```
==    equal
!=    not equal
<     less
<=    less or equal
>     greater
>=    greater or equal
```

In any comparison, the first operand must be [assignable](https://go.dev/ref/spec#Assignability) to the type of the second operand, or vice versa.

在任何比较中，第一个操作数必须可分配给第二个操作数的类型，反之亦然。

The equality operators `==` and `!=` apply to operands that are *comparable*. The ordering operators `<`, `<=`, `>`, and `>=` apply to operands that are *ordered*. These terms and the result of the comparisons are defined as follows:

相等运算符 = = 和！= 适用于具有可比性的操作数。排序运算符 < 、 < = 、 > 和 > = 适用于排序的操作数。这些术语和比较结果的定义如下:

- Boolean values are comparable. Two boolean values are equal if they are either both `true` or both `false`.

  布尔值是可比较的。

- Integer values are comparable and ordered, in the usual way.

  整数值是可比较的，并按照通常的方式排序。

- Floating-point values are comparable and ordered, as defined by the IEEE-754 standard.

  浮点值是可比的和有序的，正如 ieee-754标准所定义的。

- Complex values are comparable. Two complex values `u` and `v` are equal if both `real(u) == real(v)` and `imag(u) == imag(v)`.

  复数数值是可比较的。实部虚部都相同，复数值相同。

- String values are comparable and ordered, lexically byte-wise.

  字符串值是可比较的，并且是按字节排序的。

- Pointer values are comparable. Two pointer values are equal if they point to the same variable or if both have value `nil`. Pointers to distinct [zero-size](https://go.dev/ref/spec#Size_and_alignment_guarantees) variables may or may not be equal.

  指针值是可比较的。如果两个指针值指向同一个变量，或者两个值都为 nil，则两个指针值是相等的。指向不同`zero-size`变量的指针可能相等，也可能不相等。

  #### 疑问：同一make创建的通道，是指make传入的参数一致吗？

- Channel values are comparable. Two channel values are equal if they were created by the same call to [`make`](https://go.dev/ref/spec#Making_slices_maps_and_channels) or if both have value `nil`.

  通道值是可比较的。如果两个通道值是由同一个调用创建的，或者两个通道的值都为 nil，那么它们是相等的。

- Interface values are comparable. Two interface values are equal if they have [identical](https://go.dev/ref/spec#Type_identity) dynamic types and equal dynamic values or if both have value `nil`.

  接口类型是可比的。如果两个接口具有相同的动态类型和相同的动态值，或者两者都具有值 nil，则它们是相等的。

- A value `x` of non-interface type `X` and a value `t` of interface type `T` are comparable when values of type `X` are comparable and `X` implements `T`. They are equal if `t`'s dynamic type is identical to `X` and `t`'s dynamic value is equal to `x`.

  非接口类型`X`的值`x`和接口类型`T`的值t 时刻比较的 当`x`类型的值是可比较+`X`实现接口类型`T`。

  他们是相等  当动态类型等于`X`+t动态值等于`x`

- Struct values are comparable if all their fields are comparable. Two struct values are equal if their corresponding non-[blank](https://go.dev/ref/spec#Blank_identifier) fields are equal.

  如果所有字段都是可比较的，则结构值是可比较的。如果两个结构值对应的非空字段相等，则它们相等。

- Array values are comparable if values of the array element type are comparable. Two array values are equal if their corresponding elements are equal.

  如果数组元素类型的值是可比较的，则数组值是可比较的。如果两个数组的相应元素相等，则它们的值相等。

A comparison of two interface values with identical dynamic types causes a [run-time panic](https://go.dev/ref/spec#Run_time_panics) if values of that type are not comparable. This behavior applies not only to direct interface value comparisons but also when comparing arrays of interface values or structs with interface-valued fields.

动态类型的接口值的比较会引起panic，如果动态类型不可比较。此规范不仅适用于直接的接口值比较，还适用于接口值数组，接口值字段的结构体比较。复合情况。

#### 不可比较的情况：slice,map,funcion value

Slice, map, and function values are not comparable. However, as a special case, a slice, map, or function value may be compared to the predeclared identifier `nil`. Comparison of pointer, channel, and interface values to `nil` is also allowed and follows from the general rules above.

Slice、 map 和函数值是不可比的。但是，作为一种特殊情况，片、映射或函数值可以与预声明的标识符 nil 进行比较。还允许将指针、通道和接口值比较为 nil，并遵循上面的一般规则。

```
const c = 3 < 4            // c is the untyped boolean constant true

type MyBool bool
var x, y int
var (
	// The result of a comparison is an untyped boolean.
	// The usual assignment rules apply.
	b3        = x == y // b3 has type bool
	b4 bool   = x == y // b4 has type bool
	b5 MyBool = x == y // b5 has type MyBool
)
```

### Logical operators逻辑运算符

Logical operators apply to [boolean](https://go.dev/ref/spec#Boolean_types) values and yield a result of the same type as the operands. The right operand is evaluated conditionally.逻辑运算符应用于布尔值，并产生与操作数类型相同的结果。右操作数按条件求值。

```
&&    conditional AND    p && q  is  "if p then q else false"
||    conditional OR     p || q  is  "if p then true else q"
!     NOT                !p      is  "not p"
```

### Address operators取地址操作符

For an operand `x` of type `T`, the address operation `&x` generates a pointer of type `*T` to `x`. The operand must be *addressable*, that is, either a variable, pointer indirection, or slice indexing operation; or a field selector of an addressable struct operand; or an array indexing operation of an addressable array. As an exception to the addressability requirement, `x` may also be a (possibly parenthesized) [composite literal](https://go.dev/ref/spec#Composite_literals). If the evaluation of `x` would cause a [run-time panic](https://go.dev/ref/spec#Run_time_panics), then the evaluation of `&x` does too.

对于一个类型`T`的操作数`x`，取地址操作生成一个指向`x`的类型未`*T`的指针。这个操作数必须是可寻址，也就是说要么是一个变量，指针引用，切片索引操作，要么是一个可寻址的结构体操作数的字段选择器；要么是一个可寻址的数组的数组索引操作。

作为例外，`x`可以是复合字面量。如果`x`的计算造成`panic`那么取地址操作也会`panic`

For an operand `x` of pointer type `*T`, the pointer indirection `*x` denotes the [variable](https://go.dev/ref/spec#Variables) of type `T` pointed to by `x`. If `x` is `nil`, an attempt to evaluate `*x` will cause a [run-time panic](https://go.dev/ref/spec#Run_time_panics).对于指针类型 `* T `的操作数 `x`，指针引用`*x`表示一个变量，变量类型是`T`，变量是被 x指针 指向。如果 x 为 nil，则尝试计算 * x 将导致运行时恐慌。

```
&x
&a[f(2)]
&Point{2, 3}
*p
*pf(x)

var x *int = nil
*x   // causes a run-time panic
&*x  // causes a run-time panic
```

### Receive operator: <-ch

For an operand `ch` of [channel type](https://go.dev/ref/spec#Channel_types), the value of the receive operation `<-ch` is the value received from the channel `ch`. The channel direction must permit receive operations, and the type of the receive operation is the element type of the channel. The expression blocks until a value is available. Receiving from a `nil` channel blocks forever. 对于通道类型的操作数 ch，接收操作 `<-ch` 的值是从通道 ch 接收的值。通道方向必须允许接收操作，而接收操作的类型是通道的元素类型。表达式会阻塞，直到一个值可获得。接收来自Nil管道是永远阻塞的。

#### 对关闭管道的接收操作不会run-time panic吗？

##### 区别于12Stmt.md的sendStmt。发送给关闭管道会造成run-time panic

A receive operation on a [closed](https://go.dev/ref/spec#Close) channel can always proceed immediately, yielding the element type's [zero value](https://go.dev/ref/spec#The_zero_value) after any previously sent values have been received.关闭通道上的接收操作始终可以立即执行，在接收到任何以前发送的值之后，产生元素类型的零值。【虽然不会像发送操作那样run-time panic但是接受到的值都是元素类型的0值 】

```
v1 := <-ch
v2 = <-ch
f(<-ch)
<-strobe  // wait until clock pulse and discard received value
```

A receive expression used in an [assignment](https://go.dev/ref/spec#Assignments) or initialization of the special form接收表达式可用于特殊形式的赋值或初始化

```
x, ok = <-ch
x, ok := <-ch
var x, ok = <-ch
var x, ok T = <-ch
```

yields an additional untyped boolean result reporting whether the communication succeeded. The value of `ok` is `true` if the value received was delivered by a successful send operation to the channel, or `false` if it is a zero value generated because the channel is `closed` and empty.产生一个额外的无类型布尔结果，报告通信是否成功。如果通过成功的发送操作将接收到的值传递到通道，则 ok 的值为 true; 如果值为零，则为 false，因为通道已关闭且为空。

## 14Conversions转化

A conversion changes the [type](https://go.dev/ref/spec#Types) of an expression to the type specified by the conversion. A conversion may appear literally in the source, or it may be *implied* by the context in which an expression appears.

转换将表达式的类型更改为转换所指定的类型。转换可能按字面意思出现在源中，也可能由表达式出现的上下文所暗示。

#### 显式转换

An *explicit* conversion is an expression of the form `T(x)` where `T` is a type and `x` is an expression that can be converted to type `T`.显式转换是形如 `T (x)`的表达式，其中 `T`是一个类型，x 是一个可以转换为类型 `T`的表达式。

```
Conversion = Type "(" Expression [ "," ] ")" .
```

#### ambiguity

If the type starts with the operator `*` or `<-`, or if the type starts with the keyword `func` and has no result list, it must be parenthesized when necessary to avoid ambiguity:

如果类型以操作符 * 或 <-开始，或者类型以关键字 func 开始，没有结果列表，则必须在必要时加上括号以避免歧义:

```
*Point(p)        // same as *(Point(p))
(*Point)(p)      // p is converted to *Point
<-chan int(c)    // same as <-(chan int(c))
(<-chan int)(c)  // c is converted to <-chan int
func()(x)        // function signature func() x
(func())(x)      // x is converted to func()
(func() int)(x)  // x is converted to func() int
func() int(x)    // x is converted to func() int (unambiguous)
```

A [constant](https://go.dev/ref/spec#Constants) value `x` can be converted to type `T` if `x` is [representable](https://go.dev/ref/spec#Representability) by a value of `T`. 如果 x 可以用 类型`T` 的值表示，那么常量值 x 可以转换为类型 t

#### 数字可以转换成字符串

As a special case, an integer constant `x` can be explicitly converted to a [string type](https://go.dev/ref/spec#String_types) using the [same rule](https://go.dev/ref/spec#Conversions_to_and_from_a_string_type) as for non-constant `x`.

作为一种特殊情况，整数常量 x 可以使用与非常量 x 相同的规则显式转换为字符串类型。

Converting a constant yields a typed constant as result.转换一个常量会产生一个类型化的常量。

```
uint(iota)               // iota value of type uint
float32(2.718281828)     // 2.718281828 of type float32
complex128(1)            // 1.0 + 0.0i of type complex128
float32(0.49999999)      // 0.5 of type float32
float64(-1e-1000)        // 0.0 of type float64
string('x')              // "x" of type string
string(0x266c)           // "♬" of type string
MyString("foo" + "bar")  // "foobar" of type MyString
string([]byte{'a'})      // not a constant: []byte{'a'} is not a constant
(*int)(nil)              // not a constant: nil is not a constant, *int is not a boolean, numeric, or string type
int(1.2)                 // illegal: 1.2 cannot be represented as an int
string(65.0)             // illegal: 65.0 is not an integer constant
```

A non-constant value `x` can be converted to type `T` in any of these cases:在以下任何情况下，非常量值 `x`都可以转换为类型 `T`:

- `x` is [assignable](https://go.dev/ref/spec#Assignability) to `T`.X 可赋值给 t。

- ignoring struct tags (see below), `x`'s type and `T` have [identical](https://go.dev/ref/spec#Type_identity) [underlying types](https://go.dev/ref/spec#Types).忽略 struct 标记(见下文) ，x 的类型和 t 具有相同的底层类型。

- ignoring struct tags (see below), `x`'s type and `T` are pointer types that are not [defined types](https://go.dev/ref/spec#Type_definitions), and their pointer base types have identical underlying types.忽略结构体标记(见下文) ，`x`的类型和 `T`是指针类型，它们不是定义类型，它们的指针基类型具有相同的基础类型。

- `x`'s type and `T` are both integer or floating point types.X 的类型和 t 都是整数或浮点类型。

- `x`'s type and `T` are both complex types.类型和 t 都是复数类型。

  #### 字符串的转换

- `x` is an integer or a slice of bytes or runes and `T` is a string type.X 是一个整数或字节切片或rune符文，t 是一个字符串类型。

- `x` is a string and `T` is a slice of bytes or runes.`x` 是一个字符串，`T`是字节切片或符文。

- 

- `x` is a slice, `T` is a pointer to an array, and the slice and array types have [identical](https://go.dev/ref/spec#Type_identity) element types.

  `x`是一个 slice，`T` 是一个指向数组的指针，slice 和 array 类型具有相同的元素类型。
  
  #### 结构体标记

[Struct tags](https://go.dev/ref/spec#Struct_types) are ignored when comparing struct types for identity for the purpose of conversion:

为了转换的目的，比较结构类型的标识时忽略结构标记:

```
type Person struct {
	Name    string
	Address *struct {
		Street string
		City   string
	}
}

var data *struct {
	Name    string `json:"name"`
	Address *struct {
		Street string `json:"street"`
		City   string `json:"city"`
	} `json:"address"`
}

var person = (*Person)(data)  // ignoring tags, the underlying types are identical
```

Specific rules apply to (non-constant) conversions between numeric types or to and from a string type. These conversions may change the representation of `x` and incur a run-time cost. All other conversions only change the type but not the representation of `x`.特定规则适用于数值类型之间或与字符串类型之间的(非常数)转换。这些转换可能会改变 x 的表示形式，并产生运行时成本。所有其他转换只改变类型，而不改变 x 的表示形式。

There is no linguistic mechanism to convert between pointers and integers. The package [`unsafe`](https://go.dev/ref/spec#Package_unsafe) implements this functionality under restricted circumstances.没有语言机制来在指针和整数之间进行转换。`unsafe`包在受限制的情况下实现这个功能。

### 1Conversions between numeric types数值类型之间的转换

For the conversion of non-constant numeric values, the following rules apply:对于非常量数值的转换，适用以下规则:

1. When converting between integer types, if the value is a signed integer, it is sign extended to implicit infinite precision; otherwise it is zero extended. It is then truncated to fit in the result type's size. For example, if `v := uint16(0x10F0)`, then `uint32(int8(v)) == 0xFFFFFFF0`. The conversion always yields a valid value; there is no indication of overflow.

   在整数类型之间进行转换时，如果值是有符号整数，则将其符号扩展为隐式无限精度; 否则为零扩展。然后将其截断以适应结果类型的大小。例如，如果 v: = uint16(0x10F0) ，则 uint32(int8(v)) = = 0xFFFFFFF0。转换总是生成一个有效值; 没有溢出指示。

2. When converting a floating-point number to an integer, the fraction is discarded (truncation towards zero).

   当将浮点数转换为整数时，分数部分将被丢弃(截断向零)。

3. When converting an integer or floating-point number to a floating-point type, or a complex number to another complex type, the result value is rounded to the precision specified by the destination type. For instance, the value of a variable `x` of type `float32` may be stored using additional precision beyond that of an IEEE-754 32-bit number, but float32(x) represents the result of rounding `x`'s value to 32-bit precision. Similarly, `x + 0.1` may use more than 32 bits of precision, but `float32(x + 0.1)` does not.当将整数或浮点数转换为浮点数类型，或将复数转换为其他复数类型时，结果值将四舍五入到目标类型指定的精度。

   例如，float32类型的变量 x 的值可以使用比 ieee-75432位数更高的精度来存储，但 float32(x)表示将 x 的值舍入到32位精度的结果。类似地，x + 0.1可能使用超过32位的精度，但 float32(x + 0.1)不使用。

In all non-constant conversions involving floating-point or complex values, if the result type cannot represent the value the conversion succeeds but the result value is implementation-dependent.

在所有涉及浮点或复杂值的非常数转换中，如果结果类型不能表示值，转换会成功，但结果值依赖于实现。

### 2Conversions to and from a string type与字符串类型的转换和与字符串类型的转换

1. #### 整数-->字符串

   Converting a signed or unsigned integer value to a string type yields a string containing the UTF-8 representation of the integer. Values outside the range of valid Unicode code points are converted to 

   将有符号或无符号整数值转换为字符串类型会生成包含整数的 utf-8表示形式的字符串。有效 Unicode 代码点范围以外的值转换为

   ```
   "\uFFFD".              U+FFFD	�	占位字符（英语：Replacement Character）
   ```

   ```
   string('a')       // "a"
   string(-1)        // "\ufffd" == "\xef\xbf\xbd"
   string(0xf8)      // "\u00f8" == "ø" == "\xc3\xb8"
   type MyString string
   MyString(0x65e5)  // "\u65e5" == "日" == "\xe6\x97\xa5"
   hex ox 
   ```

2. #### 字节切片转字符串

   Converting a slice of bytes to a string type yields a string whose successive bytes are the elements of the slice.

   将字节片转换为字符串类型会产生一个字符串，其后续的字节是片的元素

   ```
   string([]byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'})   // "hellø"
   string([]byte{})                                     // ""
   string([]byte(nil))                                  // ""
   
   type MyBytes []byte
   string(MyBytes{'h', 'e', 'l', 'l', '\xc3', '\xb8'})  // "hellø"
   ```

3. #### 符文切片转字符串

   Converting a slice of runes to a string type yields a string that is the concatenation of the individual rune values converted to strings.

   将符文切片转换为字符串类型会生成一个字符串，该字符串是转换为字符串的各个符文值的串联

   ```
   string([]rune{0x767d, 0x9d6c, 0x7fd4})   // "\u767d\u9d6c\u7fd4" == "白鵬翔"
   string([]rune{})                         // ""
   string([]rune(nil))                      // ""
   
   type MyRunes []rune
   string(MyRunes{0x767d, 0x9d6c, 0x7fd4})  // "\u767d\u9d6c\u7fd4" == "白鵬翔"
   ```

4. #### 字符串转字节切片

   Converting a value of a string type to a slice of bytes type yields a slice whose successive elements are the bytes of the string. 

   将字符串类型的值转换为字节切片会产生一个切片，其连续元素是字符串的字节

   ```
   []byte("hellø")   // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
   []byte("")        // []byte{}
   MyBytes("hellø")  // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
   ```

5. #### 字符串转符文

   Converting a value of a string type to a slice of runes type yields a slice containing the individual Unicode code points of the string.将一个字符串类型的值转换为一个 runes 类型的片，将产生一个包含该字符串的单个 Unicode 代码点的片

   ```
   []rune(MyString("白鵬翔"))  // []rune{0x767d, 0x9d6c, 0x7fd4}
   []rune("")                 // []rune{}
   MyRunes("白鵬翔")           // []rune{0x767d, 0x9d6c, 0x7fd4}
   ```

### 3Conversions from slice to array pointer从切片到数组指针的转换

#### 参考取地址操作符部分:panics

Converting a slice to an array pointer yields a pointer to the underlying array of the slice. If the [length](https://go.dev/ref/spec#Length_and_capacity) of the slice is less than the length of the array, a [run-time panic](https://go.dev/ref/spec#Run_time_panics) occurs.将一个切片转换为数组指针会产生一个指向该切片的基础数组的指针。如果切片的长度小于数组的长度，则会发生运行时恐慌。

```
s := make([]byte, 2, 4)
强制类型转换[0]byte byte数组，长度0
s0 := (*[0]byte)(s)      // s0 != nil
长度一的字节数组的指针
s1 := (*[1]byte)(s[1:])  // &s1[0] == &s[1]
s2 := (*[2]byte)(s)      // &s2[0] == &s[0]
为切片的元素建立指针，现在有4个字节指针，但是切片只有2个元素，s有2个没有初始化是nil，指针无效
s4 := (*[4]byte)(s)      // panics: len([4]byte) > len(s)

var t []string
t0 := (*[0]string)(t)    // t0 == nil
t1 := (*[1]string)(t)    // panics: len([1]string) > len(t)

u := make([]byte, 0)
u0 = (*[0]byte)(u)       // u0 != nil
```

## 15Constant expressions常量表达式

Constant expressions may contain only [constant](https://go.dev/ref/spec#Constants) operands and are evaluated at compile time.

常量表达式可以只包含常量操作数，并在编译时计算。

Untyped boolean, numeric, and string constants may be used as operands wherever it is legal to use an operand of boolean, numeric, or string type, respectively.

非类型化的布尔型、数值型和字符串常量可以用作操作数，只要分别使用布尔型、数值型或字符串类型的操作数是合法的。

A constant [comparison](https://go.dev/ref/spec#Comparison_operators) always yields an untyped boolean constant. If the left operand of a constant [shift expression](https://go.dev/ref/spec#Operators) is an untyped constant, the result is an integer constant; otherwise it is a constant of the same type as the left operand, which must be of [integer type](https://go.dev/ref/spec#Numeric_types).常量比较总是产生一个无类型的布尔常量。如果常量移位表达式的左操作数是一个无类型常量，则结果是一个整数常量; 否则，它是一个与左操作数类型相同的常量，该常量必须是整数类型。

Any other operation on untyped constants results in an untyped constant of the same kind; that is, a boolean, integer, floating-point, complex, or string constant. If the untyped operands of a binary operation (other than a shift) are of different kinds, the result is of the operand's kind that appears later in this list: integer, rune, floating-point, complex. For example, an untyped integer constant divided by an untyped complex constant yields an untyped complex constant.

对非类型化常量的任何其他操作都会生成同类型的非类型化常量，即布尔型常量、整数型常量、浮点型常量、复数型常量或字符串常量。如果一个二元运算的非类型化操作数(shift 除外)是不同类型的，那么结果就是操作数类型的结果，这将在下面的列表中出现: 整数、符文、浮点数、复数。例如，一个非类型化的整数常量除以一个非类型化的复数常量，就会产生一个非类型化的复数常量。

```
const a = 2 + 3.0          // a == 5.0   (untyped floating-point constant)
const b = 15 / 4           // b == 3     (untyped integer constant)
const c = 15 / 4.0         // c == 3.75  (untyped floating-point constant)
const Θ float64 = 3/2      // Θ == 1.0   (type float64, 3/2 is integer division)
const Π float64 = 3/2.     // Π == 1.5   (type float64, 3/2. is float division)
const d = 1 << 3.0         // d == 8     (untyped integer constant)
const e = 1.0 << 3         // e == 8     (untyped integer constant)
const f = int32(1) << 33   // illegal    (constant 8589934592 overflows int32)
const g = float64(2) >> 1  // illegal    (float64(2) is a typed floating-point constant)
const h = "foo" > "bar"    // h == true  (untyped boolean constant)
const j = true             // j == true  (untyped boolean constant)
const k = 'w' + 1          // k == 'x'   (untyped rune constant)
const l = "hi"             // l == "hi"  (untyped string constant)
const m = string(k)        // m == "x"   (type string)
const Σ = 1 - 0.707i       //            (untyped complex constant)
const Δ = Σ + 2.0e-4       //            (untyped complex constant)
const Φ = iota*1i - 1/1i   //            (untyped complex constant)
```

Applying the built-in function `complex` to untyped integer, rune, or floating-point constants yields an untyped complex constant.

将内置函数复数应用于非类型化的整数、符文或浮点常数，会生成一个非类型化的复数常数。

```
const ic = complex(0, c)   // ic == 3.75i  (untyped complex constant)
const iΘ = complex(0, Θ)   // iΘ == 1i     (type complex128)
```

Constant expressions are always evaluated exactly; intermediate values and the constants themselves may require precision significantly larger than supported by any predeclared type in the language. The following are legal declarations:

常量表达式始终是精确计算的; 中间值和常量本身所需的精度可能比语言中任何预声明的类型所支持的精度大得多。以下是法律声明:

```
const Huge = 1 << 100         // Huge == 1267650600228229401496703205376  (untyped integer constant)
const Four int8 = Huge >> 98  // Four == 4                                (type int8)
```

The divisor of a constant division or remainder operation must not be zero:

常数除法或余数运算的除数不能为零:

```
3.14 / 0.0   // illegal: division by zero
```

The values of *typed* constants must always be accurately [representable](https://go.dev/ref/spec#Representability) by values of the constant type. The following constant expressions are illegal:

类型化常量的值必须始终能够精确地用常量类型的值表示。下面的常量表达式是非法的:

```
uint(-1)     // -1 cannot be represented as a uint
int(3.14)    // 3.14 cannot be represented as an int
int64(Huge)  // 1267650600228229401496703205376 cannot be represented as an int64
Four * 300   // operand 300 cannot be represented as an int8 (type of Four)
Four * 100   // product 400 cannot be represented as an int8 (type of Four)
```

The mask used by the unary bitwise complement operator `^` matches the rule for non-constants: the mask is all 1s for unsigned constants and -1 for signed and untyped constants.

一元位补运算符 ^ 使用的掩码匹配非常量的规则: 掩码全部为1表示无符号常量,-1表示有符号和无类型的常量。

```
^1         // untyped integer constant, equal to -2
uint8(^1)  // illegal: same as uint8(-2), -2 cannot be represented as a uint8
^uint8(1)  // typed uint8 constant, same as 0xFF ^ uint8(1) = uint8(0xFE)
int8(^1)   // same as int8(-2)
^int8(1)   // same as -1 ^ int8(1) = -2
```

Implementation restriction: A compiler may use rounding while computing untyped floating-point or complex constant expressions; see the implementation restriction in the section on [constants](https://go.dev/ref/spec#Constants). This rounding may cause a floating-point constant expression to be invalid in an integer context, even if it would be integral when calculated using infinite precision, and vice versa.

实现限制: 编译器在计算非类型化浮点表达式或复杂常量表达式时可能使用舍入; 请参阅关于常量的部分中的实现限制。这种舍入可能导致浮点常量表达式在整数上下文中无效，即使在使用无限精度计算时它是整数，反之亦然。

## 16Order of evaluation

At package level, [initialization dependencies](https://go.dev/ref/spec#Package_initialization) determine the evaluation order of individual initialization expressions in [variable declarations](https://go.dev/ref/spec#Variable_declarations). Otherwise, when evaluating the [operands](https://go.dev/ref/spec#Operands) of an expression, assignment, or [return statement](https://go.dev/ref/spec#Return_statements), all function calls, method calls, and communication operations are evaluated in lexical left-to-right order.

在包级别上，初始化依赖关系决定变量声明中各个初始化表达式的计算顺序。否则，在计算表达式、赋值或返回语句的操作数时，所有函数调用、方法调用和通信操作都按从左到右的词法顺序计算。

For example, in the (function-local) assignment

例如，在(function-local)赋值中

```
y[f()], ok = g(h(), i()+x[j()], <-c), k()
```

the function calls and communication happen in the order `f()`, `h()`, `i()`, `j()`, `<-c`, `g()`, and `k()`. However, the order of those events compared to the evaluation and indexing of `x` and the evaluation of `y` is not specified.

函数调用和通信按 f ()、 h ()、 i ()、 j ()、 <-c、 g ()和 k ()的顺序进行。但是，与 x 的计算和索引以及 y 的计算相比，这些事件的顺序并未指定。

```
a := 1
f := func() int { a++; return a }
x := []int{a, f()}            // x may be [1, 2] or [2, 2]: evaluation order between a and f() is not specified
m := map[int]int{a: 1, a: 2}  // m may be {2: 1} or {2: 2}: evaluation order between the two map assignments is not specified
n := map[int]int{a: f()}      // n may be {2: 3} or {3: 3}: evaluation order between the key and the value is not specified
```

At package level, initialization dependencies override the left-to-right rule for individual initialization expressions, but not for operands within each expression:在包级别，初始化依赖关系覆盖了单个初始化表达式的从左到右规则，但是不覆盖每个表达式中的操作数:

```
var a, b, c = f() + v(), g(), sqr(u()) + v()

func f() int        { return c }
func g() int        { return a }
func sqr(x int) int { return x*x }

// functions u and v are independent of all other variables and functions
```

The function calls happen in the order `u()`, `sqr()`, `v()`, `f()`, `v()`, and `g()`.

函数调用按 u ()、 sqr ()、 v ()、 f ()、 v ()和 g ()的顺序进行。

Floating-point operations within a single expression are evaluated according to the associativity of the operators. Explicit parentheses affect the evaluation by overriding the default associativity. In the expression `x + (y + z)` the addition `y + z` is performed before adding `x`.

单个表达式中的浮点运算是根据运算符的结合性来计算的。显式括号通过覆盖默认关联性来影响计算。在表达式 x + (y + z)中，加法 y + z 是在加法 x 之前进行的。