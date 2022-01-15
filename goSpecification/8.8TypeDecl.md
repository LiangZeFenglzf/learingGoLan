[TOC]

[类型属性和值]: https://go.dev/ref/spec#Properties_of_types_and_values
[类型声明]: https://go.dev/ref/spec#Type_declarations
[Go预定义类型]: https://go.dev/ref/spec#Types
[了解源码展示的token]: https://go.dev/ref/spec#Letters_and_digits	"Lexical elements/token"
[了解源码展示用的符号规则]: https://go.dev/ref/spec#Notation

## 先介绍Tokens

```

Tokens form the vocabulary of the Go language. token形成Go的词汇表
我们把写出来的程序理解为英语文章，文章就是有句子构成，句子由词汇构成，词汇就有  数字，符号，操作符，标点符号
There are four classes: identifiers, keywords, operators and punctuation, and literals.
token有四种类型： 标识符，关键字，操作符，标点符号，字面量。【这里官方文档错了把，five class】
有了这几种token 所有go的代码都可以写出来
White space, formed from spaces (U+0020), horizontal tabs (U+0009), carriage returns (U+000D), and 
newlines (U+000A), is ignored except as it separates tokens that would otherwise combine into a single token.
Newline是被忽略的2，除了 当它用作分隔token时，分隔时，newline 右边的全部  当作一个单独的token
Also, a newline or end of file may trigger the insertion of a semicolon. 触发分号插入
While breaking the input into tokens, the next token is the longest sequence of characters that form a valid token.
```

[^]: 暂时理解token就是一些符号



## Notation:关注Term

The syntax is specified using Extended Backus-Naur Form (EBNF):



```
Production  = production_name "=" [ Expression ] "." .
Expression  = Alternative { "|" Alternative } .


Alternative = Term { Term } .
              Term  = production_name | token [ "…" token ] | Group | Option | Repetition .
                                                              Group  = "(" Expression ")" .
                                                                     Option  = "[" Expression "]" .
                                                                              Repetition  = "{" Expression "}" .
```





### Production定义：

由表达式和下面的操作符构成production。 表达式是由多个term构成。

Productions are expressions constructed from terms and the following operators, in increasing precedence:

### 操作符

```
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

Lower-case production names are used to identify lexical tokens. Non-terminals are in CamelCase. Lexical tokens are enclosed in double quotes `""` or back quotes ````.

小写的production names 用来标识  字面量类型的tokens【上面提过的·五种类型之一】。不是最终的production话production name是大驼峰风格

字面量类型的token使用双引号或反引号包起来。

The form `a … b` represents the set of characters from `a` through `b` as alternatives. 

这种 a  ... b的方式 表示 一系列从a 到b之间的符号 作为alternatives部分。 alter不做翻译是因为英文更容易理解

The horizontal ellipsis `…` is also used elsewhere in the spec to informally denote various enumerations or code snippets that are not further specified. The character `…` (as opposed to the three characters `...`) is not a token of the Go language.水平省略号`…`也用于规范中的其他地方，以非正式地表示未进一步指定的各种枚举或代码片段。字符`…` （相对于三个字符`...`）不是 Go 语言的标记。



#  Type declarations

A type declaration binds an identifier, the *type name*, to a [type](https://go.dev/ref/spec#Types). 

声明形式： type   (  一个或多个typeSpec   )

Type declarations come in two forms: alias declarations and type definitions. 别名声明和类型定义

```
TypeDecl = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
TypeSpec = AliasDecl | TypeDef .
```

## 方式一：Alias declarations

An alias declaration binds an identifier to the given type. 

```
AliasDecl = identifier "=" Type .
```

Within the [scope](https://go.dev/ref/spec#Declarations_and_scope) of the identifier, it serves as an *alias* for the type.

```
type (
	nodeList = []*Node  // nodeList and []*Node are identical types
	Polar    = polar    // Polar and polar denote identical types
)
```

## 方式二（注意）：GivenType|DefinedType|UnderlyingType

A type definition creates a new, distinct type with the same [underlying type](https://go.dev/ref/spec#Types) and operations as the **given type**, and binds an identifier to it.

类型定义 语句 创建了一个新的，同 其他有着相同底层类型和相应操作 有区别 的类型  作为 我们自己给出的类型，最后绑定一个标识符给这个新类型。

其实就是 类型定义出来的就是一个全新的类型，即使基类型一致，也是不同的类型需要显式转换才能赋值。

```
TypeDef = identifier Type .  
标识符就是 given类型名也是definedl
```

The new type is called **a *defined type***. It is [different](https://go.dev/ref/spec#Type_identity) from any other type, including the type it is created from.这句话·是上面的解释 解释distinct。

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

### 关于定义类型是否继承方法的讨论

A defined type may have [methods](https://go.dev/ref/spec#Method_declarations) associated with it. It does not inherit any methods bound to the given type,定义类型可能与方法相关联。但不会继承任何 绑定给定类型 的方法

 but the [method set](https://go.dev/ref/spec#Method_sets) of an interface type or  of elements of a composite type          remains unchanged:

但是接口类型的方法集和 组合类型的元素 会不修改的保留下来也就是继承

```
// A Mutex is a data type with two methods, Lock and Unlock.
Mutex 一个有2方法的数据类型
type Mutex struct         { /* Mutex fields */ }    
func (m *Mutex) Lock()    { /* Lock implementation */ }    这里不展示具体的实现体，不是代表没有实现体
func (m *Mutex) Unlock()  { /* Unlock implementation */ }


```

#### 情况一：不继承基类型方法

```
// NewMutex has the same composition as Mutex but its method set is empty.

type NewMutex Mutex

// The method set of PtrMutex's underlying type *Mutex remains unchanged,
// but the method set of PtrMutex is empty.
```

#### 情况二：结构体内嵌字段，包含方法

```
type PtrMutex *Mutex

// The method set of *PrintableMutex contains the methods
// Lock and Unlock bound to its embedded field Mutex.
type PrintableMutex struct {
	Mutex
}
```

##### 疑问

这里的*PrintableMutex 是为了 方法接收器 吗？ 指针类型接收器。

这里说contains 是因为结构体可以embed多个类似Mutex的data type

#### 情况三：基类型是接口类型 ，完全一致

```
// MyBlock is an interface type that has the same method set as Block.
type MyBlock Block
```



Type definitions may be used to define different boolean, numeric, or string types and associate methods with them:

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
