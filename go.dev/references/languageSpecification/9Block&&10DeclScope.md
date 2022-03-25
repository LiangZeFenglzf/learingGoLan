[TOC]

# 9Blocks块

A *block* is a possibly empty sequence of declarations and statements within matching brace brackets.

块可能是匹配大括号中的声明和语句的空序列。

```
Block = "{" StatementList "}" .
StatementList = { Statement ";" } .
```

In addition to explicit blocks in the source code, there are implicit blocks:

除了源代码中的显式块，还有隐式块:

#####         universe block定义

1. The `universe block` encompasses all Go source text.

   Universe 块包含所有Go源文本。

   ##### package block定义

2. Each [package](https://go.dev/ref/spec#Packages) has a *package block* containing all Go source text for that package.

   每个包都有一个包块，其中包含该包的所有 Go 源文本。

   ##### file block定义

3. Each file has a *file block* containing all Go source text in that file.

   每个文件都有一个文件块，其中包含该文件中的所有 Go 源文本。

4. Each ["if"](https://go.dev/ref/spec#If_statements), ["for"](https://go.dev/ref/spec#For_statements), and ["switch"](https://go.dev/ref/spec#Switch_statements) statement is considered to be in its own implicit block.

   每个“ if”、“ for”和“ switch”语句都被认为位于自己的隐式块中。

5. Each clause in a ["switch"](https://go.dev/ref/spec#Switch_statements) or ["select"](https://go.dev/ref/spec#Select_statements) statement acts as an implicit block.

   “ switch”或“ select”语句中的每个子句都充当隐式块。

Blocks nest and influence [scoping](https://go.dev/ref/spec#Declarations_and_scope).

块嵌套和影响范围。

# 10Declarations and scope声明和范围

A *declaration* binds a non-[blank](https://go.dev/ref/spec#Blank_identifier) identifier to a [constant](https://go.dev/ref/spec#Constant_declarations), [type](https://go.dev/ref/spec#Type_declarations), [variable](https://go.dev/ref/spec#Variable_declarations), [function](https://go.dev/ref/spec#Function_declarations), [label](https://go.dev/ref/spec#Labeled_statements), or [package](https://go.dev/ref/spec#Import_declarations). Every identifier in a program must be declared. No identifier may be declared twice in the same block, and no identifier may be declared in both the file and package block.

声明将非空标识符绑定到常量、类型、变量、函数、标签或包。程序中的每个标识符都必须声明。不能在同一块中声明任何标识符两次，没有标识符可以既声明在文件块又声明在包块                                                                                                                                      。

The [blank identifier](https://go.dev/ref/spec#Blank_identifier) may be used like any other identifier in a declaration, but it does not introduce a binding and thus is not declared. In the package block, the identifier `init` may only be used for [`init` function](https://go.dev/ref/spec#Package_initialization) declarations, and like the blank identifier it does not introduce a new binding.

空白标识符可以像声明中的任何其他标识符一样使用，但它不引入绑定，因此不声明。在包块中，标识符` init `只能用于` init` 函数声明，并且与空白标识符一样，它不引入新的绑定。

```
Declaration   = ConstDecl | TypeDecl | VarDecl .
TopLevelDecl  = Declaration | FunctionDecl | MethodDecl .
```

The *scope* of a declared identifier is the extent of source text in which the identifier denotes the specified constant, type, variable, function, label, or package.

声明标识符的范围是源文本的范围，其中标识符表示指定的常量、类型、变量、函数、标签或包。

Go is lexically scoped using [blocks](https://go.dev/ref/spec#Blocks):

使用块来定义字典范围:

1. The scope of a [predeclared identifier](https://go.dev/ref/spec#Predeclared_identifiers) is the `universe block`.

   预先声明的标识符的范围是 universe 块。

2. The scope of an identifier denoting a constant, type, variable, or function (but not method) declared at top level (outside any function) is the `package block`.

   声明在`TopLevel`的 表示常量，类型，变量，函数（不包括方法）的`identifier`的范围是 `package block`

3. The scope of the package name of an imported package is the `file block` of the file containing the import declaration.

   导入包的包名称的范围是包含导入声明的文件的文件块。

4. The scope of an identifier denoting a method receiver, function parameter, or result variable is the `function body`.

   表示方法接收者、函数参数或结果变量的标识符的作用域是函数体。

5. The scope of `a constant or variable identifier`   declared inside a function begins at   the end of the ConstSpec or VarSpec (ShortVarDecl for short variable declarations) and ends at the end of the innermost containing block.

   ​        函数内声明的常量或变量标识符的作用域从 constanspec 或 VarSpec (短变量声明的 ShortVarDecl)的末尾开始，到最内层的包含块的末尾结束。

6. The scope of `a type identifier`                               declared inside a function begins at   the identifier in the TypeSpec and ends at the end of the innermost containing block.

   函数内声明的类型标识符的作用域从 TypeSpec 中的标识符开始，到最里面的包含块结束。

An identifier declared in a block may be `redeclared` in an inner block. While the identifier of the inner declaration is in scope, it denotes the entity declared by the `inner declaration`.

在块中声明的标识符可以在内部块中重新声明。当内部声明的标识符在范围内时，它表示由内部声明声明的实体。

#### 包子句不是声明语句

The [package clause](https://go.dev/ref/spec#Package_clause) is not a declaration; the package name does not appear in any scope. Its purpose is to identify the files belonging to the same [package](https://go.dev/ref/spec#Packages) and to specify the default package name for import declarations.

包子句不是声明; 包名称不出现在任何范围中。其目的是标识属于同一个包的文件，并为导入声明指定默认的包名。

## Label scopes标签范围

Labels are declared by [labeled statements](https://go.dev/ref/spec#Labeled_statements) and are used in the ["break"](https://go.dev/ref/spec#Break_statements), ["continue"](https://go.dev/ref/spec#Continue_statements), and ["goto"](https://go.dev/ref/spec#Goto_statements) statements. It is illegal to define a label that is never used. In contrast to other identifiers, labels are not block scoped and do not conflict with identifiers that are not labels. The scope of a label is the body of the function in which it is declared and excludes the body of any nested function.

标签由带标签的语句声明，并用于“ break”、“ continue”和“ goto”语句中。定义一个从未使用过的标签是非法的。与其他标识符不同，标识符不阻塞作用域，并且与不是标识符的标识符不冲突。标签的作用域是声明它的函数体，并在任何嵌套函数体之外                                           。

## Blank identifier空白标识符

The *blank identifier* is represented by the underscore character `_`. It serves as an anonymous placeholder instead of a regular (non-blank) identifier and has special meaning in [declarations](https://go.dev/ref/spec#Declarations_and_scope), as an [operand](https://go.dev/ref/spec#Operands), and in [assignments](https://go.dev/ref/spec#Assignments).

空白标识符由下划线字符 _ 表示。它充当匿名占位符，而不是常规(非空白)标识符，在声明、操作数和赋值中具有特殊意义。

## Predeclared identifiers预先声明的标识符

The following identifiers are implicitly declared in the` universe block`(https://go.dev/ref/spec#Blocks):

在 universe 块中隐式声明了以下标识符:

```
Types:
	bool byte complex64 complex128 error float32 float64
	int int8 int16 int32 int64 rune string
	uint uint8 uint16 uint32 uint64 uintptr

Constants:
	true false iota

Zero value:
	nil

Functions:
	append cap close complex copy delete imag len
	make new panic print println real recover
```

## Exported identifiers导出标识符

An identifier may be *exported* to permit access to it from another package. An identifier is exported if both:

可以导出一个标识符，以允许从另一个包访问该标识符。如果满足以下两点，则认为是导出标识符:

1. the first character of the identifier's name is a Unicode upper case letter (Unicode class "Lu"); and

   标识符名称的第一个字符是 Unicode 大写字母(Unicode 类“ Lu”) ; 以及

2. the identifier is declared in the` [package block](https://go.dev/ref/spec#Blocks) `or it is a [field name](https://go.dev/ref/spec#Struct_types) or [method name](https://go.dev/ref/spec#MethodName).

   标识符在包块中声明，或者是字段名或方法名。

   ##### 可导出的结构体字段名，可导出的方法名

All other identifiers are not exported.

所有其他标识符是非导出的。

## Uniqueness of identifiers标识符的唯一性

Given a set of identifiers, an identifier is called *unique* if it is *different* from every other in the set. Two identifiers are different if they are spelled differently, or if they appear in different [packages](https://go.dev/ref/spec#Packages) and are not [exported](https://go.dev/ref/spec#Exported_identifiers). Otherwise, they are the same.

给定一组标识符，如果标识符与集合中的其他标识符不同，则称其为唯一标识符。如果两个标识符的拼写不同，或者它们出现在不同的包中并且没有导出，则它们是不同的。否则，它们是一样的。

## Constant declarations常量的声明

A constant declaration binds a list of identifiers (the names of the constants) to the values of a list of [constant expressions](https://go.dev/ref/spec#Constant_expressions). The number of identifiers must be equal to the number of expressions, and the *n*th identifier on the left is bound to the value of the *n*th expression on the right.

常量声明将标识符列表(常量的名称)绑定到常量表达式列表的值。标识符的数量必须等于表达式的数量，左侧的第 n 个标识符与右侧的第 n 个表达式的值绑定。

```
ConstDecl      = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
ConstSpec      = IdentifierList [ [ Type ] "=" ExpressionList ] .

IdentifierList = identifier { "," identifier } .
ExpressionList = Expression { "," Expression } .
```

If the type is present, all constants take the type specified, and the expressions must be [assignable](https://go.dev/ref/spec#Assignability) to that type. 

##### 疑问：这句话？？

If the type is omitted, the constants take the individual types of the corresponding expressions. 



If the expression values are untyped [constants](https://go.dev/ref/spec#Constants), the declared constants remain `untyped` and the constant identifiers denote the constant values. For instance, if the expression is a floating-point literal, the constant identifier denotes a floating-point constant, even if the literal's fractional part is zero.

如果存在类型，则所有常量都采用指定的类型，表达式必须可赋值给该类型。如果省略类型，则常量取相应表达式的单个类型。如果表达式值是非类型化常量，则声明的常量将保持非类型化，常量标识符表示常量值。例如，如果表达式是浮点文字，则常量标识符表示浮点常量，即使字面量小数部分为零。

```
const Pi float64 = 3.14159265358979323846
const zero = 0.0         // untyped floating-point constant
const (
	size int64 = 1024
	eof        = -1  // untyped integer constant
)
const a, b, c = 3, 4, "foo"  // a = 3, b = 4, c = "foo", untyped integer and string constants
const u, v float32 = 0, 3    // u = 0.0, v = 3.0
```

Within a parenthesized `const` declaration list the expression list may be omitted from any but the first ConstSpec. 



Such an empty list is equivalent to the textual substitution of the first preceding non-empty expression list and its type if any. 



Omitting the list of expressions is therefore equivalent to repeating the previous list. The number of identifiers must be equal to the number of expressions in the previous list. Together with the [`iota` constant generator](https://go.dev/ref/spec#Iota) this mechanism permits light-weight declaration of sequential values:

除了第一个 ConstSpec 之外，在带括号的 const 声明列表中，表达式列表可以省略。



这样一个空列表等价于前面第一个非空表达式列表的文本替换，如果有的话，还等价于它的类型。



因此，省略表达式列表等同于重复前面的列表。标识符的数目必须等于前一个列表中的表达式的数目。该机制与 iota 常数生成器一起，允许轻量级声明顺序值:

```
const (
	Sunday = iota
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Partyday
	numberOfDays  // this constant is not exported
)
```

## Iota

Within a [constant declaration](https://go.dev/ref/spec#Constant_declarations), the predeclared identifier `iota` represents successive untyped integer [constants](https://go.dev/ref/spec#Constants). Its value is the index of the respective [ConstSpec](https://go.dev/ref/spec#ConstSpec) in that constant declaration, starting at zero. It can be used to construct a set of related constants:

在一个常量声明中，预声明的标识符 iota 表示连续的非类型整数常量。它的值是该常量声明中各个 constanspec 的索引，从零开始。它可以用来构造一组相关的常量:

```
const (
	c0 = iota  // c0 == 0
	c1 = iota  // c1 == 1
	c2 = iota  // c2 == 2
)

const (
	a = 1 << iota  // a == 1  (iota == 0)
	b = 1 << iota  // b == 2  (iota == 1)
	c = 3          // c == 3  (iota == 2, unused)
	d = 1 << iota  // d == 8  (iota == 3)
)

const (
	u         = iota * 42  // u == 0     (untyped integer constant)
	v float64 = iota * 42  // v == 42.0  (float64 constant)
	w         = iota * 42  // w == 84    (untyped integer constant)
)

const x = iota  // x == 0
const y = iota  // y == 0
```

By definition, multiple uses of `iota` in the same ConstSpec all have the same value:

根据定义，在相同的 constanspec 中多次使用 iota 都具有相同的值:

```
const (
	bit0, mask0 = 1 << iota, 1<<iota - 1  // bit0 == 1, mask0 == 0  (iota == 0)
	bit1, mask1                           // bit1 == 2, mask1 == 1  (iota == 1)
	_, _                                  //                        (iota == 2, unused)
	bit3, mask3                           // bit3 == 8, mask3 == 7  (iota == 3)
)
```

This last example exploits the [implicit repetition](https://go.dev/ref/spec#Constant_declarations) of the last non-empty expression list.

最后一个示例利用了最后一个非空表达式列表的隐式重复。

## Type declarations类型声明

A type declaration binds an identifier, the *type name*, to a [type](https://go.dev/ref/spec#Types). Type declarations come in two forms: alias declarations and type definitions.

类型声明将标识符(类型名称)绑定到类型。类型声明有两种形式: 别名声明和类型定义。

```
TypeDecl = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
TypeSpec = AliasDecl | TypeDef .
```

#### Alias declarations别名声明

An alias declaration binds an identifier to the given type.

别名声明将标识符绑定到给定的类型。

```
AliasDecl = identifier "=" Type .
```

Within the [scope](https://go.dev/ref/spec#Declarations_and_scope) of the identifier, it serves as an *alias* for the type.

在标识符的范围内，它充当类型的别名。

```
type (
	nodeList = []*Node  // nodeList and []*Node are identical types
	Polar    = polar    // Polar and polar denote identical types
)
```

#### Type definitions类型定义

A type definition creates a new, distinct type with the same [underlying type](https://go.dev/ref/spec#Types) and operations as the given type, and binds an identifier to it.

类型定义创建具有与给定类型相同的基础类型和操作的新的不同类型，并将标识符绑定到该类型。

```
TypeDef = identifier Type .
```

The new type is called a *defined type*. It is [different](https://go.dev/ref/spec#Type_identity) from any other type, including the type it is created from.

新类型称为已定义类型。它不同于任何其他类型，包括它所创建的类型。

```
type (
	Point struct{ x, y float64 }  // Point and struct{ x, y float64 } are different types
	polar Point                   // polar and Point denote different types
)

type TreeNode struct {
	left, right *TreeNode
	value *Comparable
}

type Block interface {
	BlockSize() int
	Encrypt(src, dst []byte)
	Decrypt(src, dst []byte)
}
```

A defined type may have [methods](https://go.dev/ref/spec#Method_declarations) associated with it. It does not inherit any methods bound to the given type, but the [method set](https://go.dev/ref/spec#Method_sets) of an interface type or of elements of a composite type remains unchanged:

定义的类型可能具有与其关联的方法。它不继承任何绑定到给定类型的方法，但接口类型或复合类型元素的方法集保持不变:

```
// A Mutex is a data type with two methods, Lock and Unlock.
type Mutex struct         { /* Mutex fields */ }
func (m *Mutex) Lock()    { /* Lock implementation */ }
func (m *Mutex) Unlock()  { /* Unlock implementation */ }

// NewMutex has the same composition as Mutex but its method set is empty.
type NewMutex Mutex

// The method set of PtrMutex's underlying type *Mutex remains unchanged,
// but the method set of PtrMutex is empty.
type PtrMutex *Mutex

// The method set of *PrintableMutex contains the methods
// Lock and Unlock bound to its embedded field Mutex.
type PrintableMutex struct {
	Mutex
}

// MyBlock is an interface type that has the same method set as Block.
type MyBlock Block
```

Type definitions may be used to define different boolean, numeric, or string types and associate methods with them:

类型定义可用于定义不同的布尔类型、数值类型或字符串类型，并将方法与它们关联:

```
type TimeZone int

const (
	EST TimeZone = -(5 + iota)
	CST
	MST
	PST
)

func (tz TimeZone) String() string {
	return fmt.Sprintf("GMT%+dh", tz)
}
```

## Variable declarations变量声明

### var变量声明给了initial value

A variable declaration creates one or more [variables](https://go.dev/ref/spec#Variables), binds corresponding identifiers to them, and gives each a type and an initial value.

变量声明创建一个或多个变量，将相应的标识符绑定到这些变量，并为每个变量提供一个类型和初始值。

```
VarDecl     = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
VarSpec     = IdentifierList ( Type [ "=" ExpressionList ] | "=" ExpressionList ) .
var i int
var U, V, W float64
var k = 0
var x, y float32 = -1, -2
var (
	i       int
	u, v, s = 2.0, 3.0, "bar"
)
var re, im = complexSqrt(-1)
var _, found = entries[name]  // map lookup; only interested in "found"
```

#### initial value = 表达式evaluate的值| 类型对应的zeroed vaule

If a list of expressions is given, the variables are initialized with the expressions following the rules for [assignments](https://go.dev/ref/spec#Assignments). Otherwise, each variable is initialized to its [zero value](https://go.dev/ref/spec#The_zero_value).

如果给定了表达式列表，则按照赋值规则使用表达式初始化变量。否则，将每个变量初始化为其零值。

##### 携带type的声明

If a type is present, each variable is given that type. Otherwise, each variable is given the type of the corresponding initialization value in the assignment. 

如果存在类型，则为每个变量提供该类型。否则，每个变量都会在赋值中给出相应初始化值的类型。

##### untyped constant

If that value is an untyped constant, it is first implicitly [converted](https://go.dev/ref/spec#Conversions) to its [default type](https://go.dev/ref/spec#Constants); if it is an untyped boolean value, it is first implicitly converted to type `bool`. 如果该值是非类型化常量，则首先将其隐式转换为默认类型; 如果该值是非类型化布尔值，则首先将其隐式转换为 bool 类型。

##### nil不能初始化没有显式类型的变量：有些类型的zero value不是Nil

The predeclared value `nil` cannot be used to initialize a variable with no explicit type.预声明的值 nil 不能用于初始化没有显式类型的变量。

```
var d = math.Sin(0.5)  // d is float64
var i = 42             // i is int
var t, ok = x.(T)      // t is T, ok is bool
var n = nil            // illegal
```

Implementation restriction: A compiler may make it illegal to declare a variable inside a [function body](https://go.dev/ref/spec#Function_declarations) if the variable is never used.

实现限制: 如果一个变量从未被使用，编译器可能会使在函数体中声明该变量是非法的。

## Short variable declarations短变量声明

A *short variable declaration* uses the syntax:

一个简短的变量声明使用了以下语法:

```
ShortVarDecl = IdentifierList ":=" ExpressionList .
```

It is shorthand for a regular [variable declaration](https://go.dev/ref/spec#Variable_declarations) with initializer expressions but no types:

它简化了带有初始化器表达式的正则变量声明，但没有表明类型:

```
"var" IdentifierList = ExpressionList .
i, j := 0, 10
f := func() int { return 7 }
ch := make(chan int)
r, w, _ := os.Pipe()  // os.Pipe() returns a connected pair of Files and an error, if any
_, y, _ := coord(p)   // coord() returns three values; only interested in y coordinate
```

##### 短变量声明：至少有一个新变量，旧变量redeclaration

Unlike regular variable declarations, a short variable declaration may *redeclare* variables provided they were originally declared earlier in the same block (or the parameter lists if the block is the function body) with the same type, and at least one of the non-[blank](https://go.dev/ref/spec#Blank_identifier) variables is new. 与常规变量声明不同，短变量声明可以重新声明变量，前提是它们最初是在同一块中声明的(或者如果块是函数体，参数列表) ，并且具有相同的类型，而且至少有一个非空变量是新的。

##### 重新声明：不是重新声明变量而是重新赋值

As a consequence, redeclaration can only appear in a multi-variable short declaration. Redeclaration does not introduce a new variable; it just assigns a new value to the original.

因此，重新声明只能出现在多变量短声明中。重新声明的那部分没有不引入新变量; 它只是将一个新值赋给原始变量。

```
field1, offset := nextField(str, 0)
field2, offset := nextField(str, offset)  // redeclares offset
a, a := 1, 2                              // illegal: double declaration of a or no new variable if a was declared elsewhere
```

Short variable declarations may appear only inside functions. In some contexts such as the initializers for ["if"](https://go.dev/ref/spec#If_statements), ["for"](https://go.dev/ref/spec#For_statements), or ["switch"](https://go.dev/ref/spec#Switch_statements) statements, they can be used to declare local temporary variables.

短变量声明可能只出现在函数内部。在某些上下文中，例如“ if”、“ for”或“ switch”语句的初始化器，可以使用它们声明局部临时变量。

## Function declarations函数声明

A function declaration binds an identifier, the *function name*, to a function.

函数声明将一个标识符(函数名)绑定到一个函数。

```
FunctionDecl = "func" FunctionName Signature [ FunctionBody ] .
FunctionName = identifier .
FunctionBody = Block .
```

If the function's [signature](https://go.dev/ref/spec#Function_types) declares result parameters, the function body's statement list must end in a [terminating statement](https://go.dev/ref/spec#Terminating_statements).

如果函数的签名声明结果参数，函数体的语句列表必须以`终止语句`结束。

```
func IndexRune(s string, r rune) int {
	for i, c := range s {
		if c == r {
			return i
		}
	}
	// invalid: missing return statement
}
```

A function declaration may omit the body. Such a declaration provides the signature for a function implemented outside Go, such as an assembly routine.

函数声明可能省略正文。这样的声明为在Go之外实现的函数(如 汇编routine)提供签名。

```
func min(x int, y int) int {
	if x < y {
		return x
	}
	return y
}

func flushICache(begin, end uintptr)  // implemented externally
```

## Method declarations方法声明

A method is a [function](https://go.dev/ref/spec#Function_declarations) with a *receiver*. A method declaration binds an identifier, the *method name*, to a method, and associates the method with the receiver's *base type*.

方法是带有接收器的函数。方法声明将标识符(方法名)绑定到方法，并将该方法与接收方的基类型关联。

```
MethodDecl = "func" Receiver MethodName Signature [ FunctionBody ] .
Receiver   = Parameters .
```

### 接收器不一定要是结构体

The receiver is specified via an extra parameter section preceding the method name. That parameter section must declare a single non-variadic parameter, the receiver. Its type must be a [defined](https://go.dev/ref/spec#Type_definitions) type `T` or a pointer to a defined type `T`. `T` is called the receiver *base type*. 

接收器是通过方法名称前面的一个额外参数部分指定的。`receiver`参数部分必须声明一个非可变参数。它的类型必须是已定义的类型 t，或者指向已定义类型 t 的指针. `T` 称为接收器基类型。

#### 接收器类型不能为接口类型的指针

A receiver base type cannot be a pointer or interface type and it must be defined in the same package as the method. 

The method is said to be *bound* to its receiver base type and the method name is visible only within [selectors](https://go.dev/ref/spec#Selectors) for type `T` or `*T`.

接收器基类型不能是指针或接口类型，它必须在与方法相同的包中定义。该方法被称为绑定到其接收方基类型，并且方法名称仅在类型 t 或 * t 的选择器中可见。

#### 方法签名种标识符的唯一性

A non-[blank](https://go.dev/ref/spec#Blank_identifier) receiver identifier must be [unique](https://go.dev/ref/spec#Uniqueness_of_identifiers) in the method signature. 

#### unused的接收器 ，`identifier`是可以省略的

If the receiver's value is not referenced inside the body of the method, its identifier may be omitted in the declaration. 

The same applies in general to parameters of functions and methods.

非空接收方标识符在方法签名中必须是唯一的。如果方法体内没有引用接收方的值，则可以在声明中省略其标识符。这同样适用于函数和方法的参数。

#### 结构体接收器 字段名和方法名不能相同，发生冲突了

For a base type, the non-blank names of methods bound to it must be unique. If the base type is a [struct type](https://go.dev/ref/spec#Struct_types), the non-blank method and field names must be distinct.

对于基类型，绑定到它的方法的非空名称必须是唯一的。如果基类型是结构类型，则非空方法和字段名称必须不同。



Given defined type `Point`, the declarations

给定定义的类型 Point，声明

```
func (p *Point) Length() float64 {
	return math.Sqrt(p.x * p.x + p.y * p.y)
}

func (p *Point) Scale(factor float64) {
	p.x *= factor
	p.y *= factor
}
```

bind the methods `Length` and `Scale`, with receiver type `*Point`, to the base type `Point`.

将方法 Length 和 Scale 与 receiver type * Point 绑定到基类型 Point。

### 方法是特别的函数

The type of a method is the type of a function with the receiver as first argument. For instance, the method `Scale` has type

方法的类型是以接收方为第一个参数的函数类型。例如，Scale 方法具有

```
func(p *Point, factor float64)
```

However, a function declared this way is not a method.

但是，以这种方式声明的函数不是方法。

