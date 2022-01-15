[TOC]

# Statements声明

Statements control execution.语句控制执行。

```
Statement =
	Declaration | LabeledStmt | SimpleStmt |
	GoStmt | ReturnStmt | BreakStmt | ContinueStmt | GotoStmt |
	FallthroughStmt | Block | IfStmt | SwitchStmt | SelectStmt | ForStmt |
	DeferStmt .

SimpleStmt = EmptyStmt | ExpressionStmt | SendStmt | IncDecStmt | Assignment | ShortVarDecl .
```

## 1Terminating statements终止语句

### 作用：阻止同Block的后续语句执行

A *terminating statement* prevents execution of all statements that lexically appear after it in the same [block](https://go.dev/ref/spec#Blocks). The following statements are terminating:

终止语句阻止执行在同一块中以词法形式出现在它之后的所有语句。下列声明终止:

1. A "return" or "goto" statement.

2. A call to the built-in function`panic`.

3. A block in which the statement list ends in a terminating statement.

4. An "if" statement

    in which:

   - the "else" branch is present, and
   - both branches are terminating statements.

5. A "for" statement

   in which:

   - there are no "break" statements referring to the "for" statement, and
   - the loop condition is absent.

6. A "switch" statement

   in which:

   - there are no "break" statements referring to the "switch" statement,
   - there is a default case, and
   - the statement lists in each case, including the default, end in a terminating statement, or a possibly labeled ["fallthrough" statement](https://go.dev/ref/spec#Fallthrough_statements).

7. A "select" statement

   in which:

   - there are no "break" statements referring to the "select" statement, and
   - the statement lists in each case, including the default if present, end in a terminating statement.

8. A [labeled statement](https://go.dev/ref/spec#Labeled_statements) labeling a terminating statement.

All other statements are not terminating.

A [statement list](https://go.dev/ref/spec#Blocks) ends in a terminating statement if the list is not empty and its final non-empty statement is terminating.

## 2Empty statements

The empty statement does nothing.

```
EmptyStmt = .
```

## 3Labeled statements标记语句

A labeled statement may be the target of a `goto`, `break` or `continue` statement.

带标签的语句可能是 goto、 break 或 continue 语句的目标。

```
LabeledStmt = Label ":" Statement .
Label       = identifier .
Error: log.Panic("error encountered")
```

## 4Expression statements表达式语句

## 疑问：什么是stmt context?

With the exception of specific built-in functions, function and method [calls](https://go.dev/ref/spec#Calls) and [receive operations](https://go.dev/ref/spec#Receive_operator) can appear in statement context. Such statements may be parenthesized.除了特定的内置函数之外，函数、方法调用和接收操作都可以出现在语句上下文中。这些语句可以用括号括起来。

```
ExpressionStmt = Expression .
```

```
The following built-in functions are not permitted in statement context:在语句上下文中不允许使用下列内置函数:

append cap complex imag len make new real
unsafe.Add unsafe.Alignof unsafe.Offsetof unsafe.Sizeof unsafe.Slice
```

```
h(x+y)
f.Close()
<-ch
(<-ch)
len("foo")  // illegal if len is the built-in function
```

## 5IncDec statements

### Increment Decrease自增自减声明

The "++" and "--" statements increment or decrement their operands by the untyped [constant](https://go.dev/ref/spec#Constants) `1`. As with an assignment, the operand must be [addressable](https://go.dev/ref/spec#Address_operators) or a map index expression.

“ + +”和“ --”语句将其操作数递增或递减为非类型化常量1。与赋值一样，操作数必须是可寻址的或映射索引表达式。

```
IncDecStmt = Expression ( "++" | "--" ) .
```

The following [assignment statements](https://go.dev/ref/spec#Assignments) are semantically equivalent:

下面的赋值语句在语义上是等价的:

```
IncDec statement    Assignment
x++                 x += 1
x--                 x -= 1
```

## 6Assignments

```
Assignment = ExpressionList assign_op ExpressionList .

assign_op = [ add_op | mul_op ] "=".
```

### 6.1左操作数：可寻址，map索引表达式，赋值用的空白标识符

Each left-hand side operand must be [addressable](https://go.dev/ref/spec#Address_operators), a map index expression, or (for `=` assignments only) the [blank identifier](https://go.dev/ref/spec#Blank_identifier). Operands may be parenthesized.每个左侧操作数必须是可寻址的、映射索引表达式或(仅用于 = assignment重新赋值的)空白标识符。操作数可以被括号()括起来。

```
x = 1
*p = f()
a[i] = 23
(k) = <-ch  // same as: k = <-ch
```

### 6.2assignment operation定义: `x` op= `y`      x不能是blank identifier

An *assignment operation* `x` *op*`=` `y` where *op* is a binary [arithmetic operator](https://go.dev/ref/spec#Arithmetic_operators) is equivalent to `x` `=` `x` *op* `(y)` but evaluates `x` only once. The` op=` construct is a single token. In assignment operations, both the left- and right-hand expression lists must contain exactly one single-valued expression, and the left-hand expression must not be the blank identifier.一个赋值运算 x op = y，其中 op 是一个二元算术运算符，这等价于 x = x op (y) ，但只计算一次 x。Op = 构造是一个单一的令牌。在赋值操作中，左表达式列表和右表达式列表必须恰好包含一个单值表达式，左表达式不能是空白标识符。

```
a[i] <<= 2
i &^= 1<<n
```

### 6.3tuple【元组】赋值：

#### 6.3.1form one:单个多返回值表达式

A tuple assignment assigns the individual elements of a multi-valued operation to a list of variables. There are two forms. In the first, the right hand operand is` a single multi-valued expression` such as a function call, a [channel](https://go.dev/ref/spec#Channel_types) or [map](https://go.dev/ref/spec#Map_types) operation, or a [type assertion](https://go.dev/ref/spec#Type_assertions). The number of operands on the left hand side must match the number of values. For instance, if `f` is a function returning two values,

元组赋值将多值操作的单个元素赋给变量列表。有两种形式。在第一种方法中，右操作数是单个多值表达式，如函数调用、通道或映射操作或类型断言。左边的操作数数目必须与值的数目相匹配。例如，如果 f 是一个返回两个值的函数,

```
x, y = f()
```

assigns the first value to `x` and the second to `y`. 

#### 6.3.2form two:最规矩的，一对一赋值

In the second form, the number of operands on the left must equal the number of expressions on the right, each of which must be single-valued, and the *n*th expression on the right is assigned to the *n*th operand on the left:

将第一个值赋给 x，第二个赋给 y。在第二种形式中，左边的操作数数目必须等于右边的表达式数目，每个表达式必须是单值的，右边的第 n 个表达式分配给左边的第 n 个操作数:

```
one, two, three = '一', '二', '三'
```

The [blank identifier](https://go.dev/ref/spec#Blank_identifier) provides a way to ignore right-hand side values in an assignment:

空白标识符提供了一种在赋值中忽略右边值的方法:

```
_ = x       // evaluate x but ignore it
x, _ = f()  // evaluate f() but ignore second result value
```

### 6.4赋值分2步：参考11expr.md的 selector部分

The assignment proceeds in two phases. First, the operands of [index expressions](https://go.dev/ref/spec#Index_expressions) and [pointer indirections](https://go.dev/ref/spec#Address_operators) (including implicit pointer indirections in [selectors](https://go.dev/ref/spec#Selectors)) on the left and the expressions on the right are all [evaluated in the usual order](https://go.dev/ref/spec#Order_of_evaluation). 

Second, the assignments are carried out in left-to-right order.

分配工作分两个阶段进行。首先，左侧的索引表达式和指针引用(包括隐式的指针引用selector)的操作数和右侧的表达式都按照通常的顺序计算。其次，赋值是按照从左到右的顺序进行的。

```
a, b = b, a  // exchange a and b

x := []int{1, 2, 3}
i := 0
i, x[i] = 1, 2  // set i = 1, x[0] = 2

i = 0
x[i], i = 2, 1  // set x[0] = 2, i = 1

x[0], x[0] = 1, 2  // set x[0] = 1, then x[0] = 2 (so x[0] == 2 at end)

x[1], x[3] = 4, 5  // set x[1] = 4, then panic setting x[3] = 5.

type Point struct { x, y int }
var p *Point          
x[2], p.x = 6, 7  // set x[2] = 6, then panic setting p.x = 7         隐式指针引用selector

i = 2
x = []int{3, 5, 7}
for i, x[i] = range x {  // set i, x[2] = 0, x[0]
	break
}
// after this loop, i == 0 and x == []int{3, 5, 3}
```

In assignments, each value must be [assignable](https://go.dev/ref/spec#Assignability) to the type of the operand to which it is assigned, with the following special cases:

在转让中，每个值都必须可以分配给指派给它的操作数的类型，特殊情况如下:

1. Any typed value may be assigned to the blank identifier.

   任何类型化的值都可以分配给空白标识符。
2. If an untyped constant is assigned to a variable of interface type or the blank identifier, the constant is first implicitly [converted](https://go.dev/ref/spec#Conversions) to its [default type](https://go.dev/ref/spec#Constants).

   如果将非类型化常量分配给接口类型的变量或空白标识符，则该常量首先隐式转换为其默认类型。
3. If an untyped boolean value is assigned to a variable of interface type or the blank identifier, it is first implicitly converted to type `bool`.

   如果将非类型布尔值分配给接口类型的变量或空白标识符，则首先将其隐式转换为类型 bool。

## 7没什么好说的If statements

"If" statements specify the conditional execution of two branches according to the value of a boolean expression. If the expression evaluates to true, the "if" branch is executed, otherwise, if present, the "else" branch is executed.

“ If”语句根据布尔表达式的值指定两个分支的条件执行。如果表达式的计算结果为 true，则执行“ If”分支，否则，如果存在，则执行“ else”分支。

```
IfStmt = "if" [ SimpleStmt ";" ] Expression Block [ "else" ( IfStmt | Block ) ] .
if x > max {
	x = max
}
```

The expression may be preceded by a simple statement, which executes before the expression is evaluated.

表达式的前面可以有一个简单的语句，该语句在表达式求值之前执行。

```
if x := f(); x < y {
	return x
} else if x > z {
	return z
} else {
	return y
}
```

## 8Switch statements开关语句

"Switch" statements provide multi-way execution. An expression or type is compared to the "cases" inside the "switch" to determine which branch to execute.“ Switch”语句提供多路执行。表达式或类型与“开关”中的“ cases”相比较，以确定要执行哪个分支。

```
SwitchStmt = ExprSwitchStmt | TypeSwitchStmt .
```

There are two forms: expression switches and type switches. In an expression switch, the cases contain expressions that are compared against the value of the switch expression. In a type switch, the cases contain types that are compared against the type of a specially annotated switch expression. 有两种形式: 表达式开关和类型开关。在表达式开关中，case包含与exprSwitch的值进行比较的expr。在TypeSwitch中，case包含type与专门注释的switchExpr的类型进行比较。

### 只计算一次 switchExpr

The switch expression is evaluated exactly once in a switch statement.在 switch 语句中，开关表达式只计算一次。

### 1Expression switches表达式开关

```
ExprSwitchStmt = "switch"             [ SimpleStmt ";" ] [ Expression ]              "{" { ExprCaseClause } "}" .
                                           smipleStmt比expression先执行

ExprCaseClause = ExprSwitchCase ":" StatementList .
                 ExprSwitchCase = "case" ExpressionList | "default" .
```

In an expression switch, the switch expression is evaluated and the case expressions, which need not be constants, are evaluated left-to-right and top-to-bottom; 在exprSwitch中，这个switch expr被计算，case表达式【不一定是常量，不做要求】，从左到右，从上到下计算

#### 1.1重点：第一个equal执行，其余忽略

the first one that equals the switch expression triggers execution of the statements of the associated case; the other cases are skipped.第一个同switchExpr相等的case触发执行 相关case的stmt。同时，其余的case是被忽略的

 If no case matches and there is a "default" case, its statements are executed. There can be at most one default case and it may appear anywhere in the "switch" statement. 如果没有匹配的case，如果有默认case，就执行默认case的stmt。在一个switchStmt中最多只有一个`default`case，这个默认case可以出现在switch的任何地方A missing switch expression is equivalent to the boolean value `true`.在一个switchStmt中如果缺少 switchExpr，认为和真值`true`相等

缺少的开关表达式等价于布尔值 true。

#### 1.2 convert untyped ..

If the switch expression evaluates to an untyped constant, it is first implicitly [converted](https://go.dev/ref/spec#Conversions) to its [default type](https://go.dev/ref/spec#Constants). The predeclared untyped value `nil` cannot be used as a switch expression. The switch expression type must be [comparable](https://go.dev/ref/spec#Comparison_operators).

如果开关表达式的计算结果为非类型常数，则首先将其隐式转换为其默认类型。预先声明的非类型化值 nil 不能用作开关表达式。开关表达式类型必须具有可比性。

If a case expression is untyped, it is first implicitly [converted](https://go.dev/ref/spec#Conversions) to the type of the switch expression. For each (possibly converted) case expression `x` and the value `t` of the switch expression, `x == t` must be a valid [comparison](https://go.dev/ref/spec#Comparison_operators).

如果case表达式是非类型化的，则首先将其隐式转换为开关表达式的类型。对于每个(可能已转换的)大小写表达式 x 和开关表达式的值 t，x = = t 必须是一个有效的比较。

#### 1.3重点：switch expr的作用：如果是decl和init临时变量，那么用来同case expr作比较；

##### 注意不和default比较

In other words, the switch expression is treated as if it were used to declare and initialize a temporary variable `t` without explicit type; it is that value of `t` against which each case expression `x` is tested for equality.

换句话说，开关表达式被当作是用来声明和初始化临时变量 t，而不是显式类型; 是用 t 的值，去比较每个`case`表达式 x  ， 测试这两是不是相等。

#### 1.4流程运行

In a case or default clause, the last non-empty statement may be a (possibly [labeled](https://go.dev/ref/spec#Labeled_statements)) ["fallthrough" statement](https://go.dev/ref/spec#Fallthrough_statements) to indicate that control should flow from the end of this clause to the first statement of the next clause. Otherwise control flows to the end of the "switch" statement. 

在 case 或 default 子句中，最后一个非空`statement`可能是(可能标记为)“ fallthrough”语句，以指示控制应该从该子句的结尾流向下一个子句的第一个语句。否则，控制流直接将流到“ switch”语句的末尾【也就是结束整个switch语句，而不是第一种情况的结束当前case去下一个case比对case expression` x`】。“ A 

##### 2.4.1 fallthrough出现的位置：不在最后一个`ExprCaseClause`,出现在`ExprCaseClause`的·StatementList的最后一个Statement

"fallthrough" statement may appear as the last statement of all but the last clause of an expression switch.

fallthrough”语句可能作为除switch的最后一个子句以外的所有子句的最后一个语句出现。

#### 1.5重点Simple Stmt比swith expr先执行

The switch expression may be preceded by` a simple statement`, which executes before the expression is evaluated.

在开关表达式之前可以有一个执行一个简单的语句，该语句在表达式求值之前执行。

```
switch tag {  tag是switch expression  
default: s3()
case 0, 1, 2, 3: s1()
case 4, 5, 6, 7: s2()
}
失去 switch expr的情况 means `switch x := f();true`
首先不满足2.3 因为这里的switch expression是empty,不是用来decl和init变量
下面的·这个switch类似if-else语句
switch x := f(); {  // missing switch expression means "true"
case x < 0: return -x
default: return x
}

switch {  没有简单声明 也没有expression   直接进入case
case x < y: f1()
case x < z: f2()
case x == 4: f3()
}
```

Implementation restriction: A compiler may disallow multiple case expressions evaluating to the same constant. For instance, the current compilers disallow duplicate integer, floating point, or string constants in case expressions.

实现限制: 编译器可能禁止计算到同一常量的多个大小写表达式。例如，当前编译器禁止大小写表达式中的重复整数、浮点数或字符串常量。

### 2Type switches类型开关

A type switch compares types rather than values. It is otherwise similar to an expression switch. It is marked by a special switch expression that has the form of a [type assertion](https://go.dev/ref/spec#Type_assertions) using the keyword `type` rather than an actual type:

类型开关比较的是类型而不是值。它在其他方面类似于表达式开关。它由一个特殊的 switch 表达式标记，该表达式的形式是使用`type`关键字而不是实际类型的    类型断言:

```
switch x.(type) {
// cases
}
```

Cases then match actual types `T` against the dynamic type of the expression `x`. As with type assertions, `x` must be of [interface type](https://go.dev/ref/spec#Interface_types), and each `non-interface type` `T` listed in a case must implement the type of `x`. The types listed in the cases of a type switch must all be [different](https://go.dev/ref/spec#Type_identity).

然后，用例将实际类型 t 与表达式 x 的动态类型匹配。与类型断言一样，x 必须是接口类型，并且案例中列出的每个非接口类型 t 都必须实现 x 的类型。类型开关的例子中列出的类型必须都是不同的。

```
TypeSwitchStmt  = "switch" [ SimpleStmt ";" ] TypeSwitchGuard "{" { TypeCaseClause } "}" .
TypeSwitchGuard = [ identifier ":=" ] PrimaryExpr "." "(" "type" ")" .
TypeCaseClause  = TypeSwitchCase ":" StatementList .
TypeSwitchCase  = "case" TypeList | "default" .
TypeList        = Type { "," Type } .
```

The TypeSwitchGuard may include a [short variable declaration](https://go.dev/ref/spec#Short_variable_declarations). When that form is used, the variable is declared at the end of the TypeSwitchCase in the [implicit block](https://go.dev/ref/spec#Blocks) of each clause. In clauses with a case listing exactly one type, the variable has that type; otherwise, the variable has the type of the expression in the TypeSwitchGuard.

TypeSwitchGuard 可能包括一个简短的变量声明。当使用该这种形式时，变量在每个子句的隐式块中的 TypeSwitchCase 结尾声明。在只列出一种类型的 case 子句中，变量具有该类型; 否则，变量具有 TypeSwitchGuard 中表达式的类型。

Instead of a type, a case may use the predeclared identifier [`nil`](https://go.dev/ref/spec#Predeclared_identifiers); that case is selected when the expression in the TypeSwitchGuard is a `nil` interface value. There may be at most one `nil` case.

Case 可以使用预声明的标识符 nil，而不是类型; 当 TypeSwitchGuard 中的表达式为 nil 接口值时，就会选择这个 case。最多可能有一个nil case。

Given an expression `x` of type `interface{}`, the following type switch:

给定一个接口{}类型的表达式 x，下面的类型切换:

```
switch i := x.(type) {
case nil:
	printString("x is nil")                // type of i is type of x (interface{})
case int:
	printInt(i)                            // type of i is int
case float64:
	printFloat64(i)                        // type of i is float64
case func(int) float64:
	printFunction(i)                       // type of i is func(int) float64
case bool, string:
	printString("type is bool or string")  // type of i is type of x (interface{})
default:
	printString("don't know the type")     // type of i is type of x (interface{})
}
```

could be rewritten:

可以重写:

```
v := x  // x is evaluated exactly once
if v == nil {
	i := v                                 // type of i is type of x (interface{})
	printString("x is nil")
} else if i, isInt := v.(int); isInt {
	printInt(i)                            // type of i is int
} else if i, isFloat64 := v.(float64); isFloat64 {
	printFloat64(i)                        // type of i is float64
} else if i, isFunc := v.(func(int) float64); isFunc {
	printFunction(i)                       // type of i is func(int) float64
} else {
	_, isBool := v.(bool)
	_, isString := v.(string)
	if isBool || isString {
		i := v                         // type of i is type of x (interface{})
		printString("type is bool or string")
	} else {
		i := v                         // type of i is type of x (interface{})
		printString("don't know the type")
	}
}
```

The type switch guard may be preceded by a simple statement, which executes before the guard is evaluated.

在类型开关保护之前可以执行一个简单的语句，该语句在guard计算之前执行。

The "fallthrough" statement is not permitted in a type switch.类型开关中不允许使用“ fallthrough”语句。

## 9For statements for迭代

A "for" statement specifies repeated execution of a block. There are three forms: The iteration may be controlled by a single condition, a "for" clause, or a "range" clause.

“ for”语句指定重复执行一个块。有三种形式: 迭代可以由单个条件、“ for”子句或“ range”子句控制。

```
ForStmt = "for" [ Condition | ForClause | RangeClause ] Block .
Condition = Expression.
```

### 9.1For statements with single condition单个条件

In its simplest form, a "for" statement specifies the repeated execution of a block as long as a boolean condition evaluates to true. The condition is evaluated before each iteration. If the condition is absent, it is equivalent to the boolean value `true`.

在其最简单的形式中，“ for”语句指定一个块的重复执行，只要布尔条件的计算结果为 true。条件在每次迭代之前进行评估。如果条件不存在，则等效于布尔值 true。

```
for a < b {
	a *= 2
}
```

### 9.2For statements with `for` clause 带 For 子句

A "for" statement with a ForClause is also controlled by its condition, but additionally it may specify an *init* and a *post* statement, such as an assignment, an increment or decrement statement. The init statement may be a [short variable declaration](https://go.dev/ref/spec#Short_variable_declarations), but the post statement must not. Variables declared by the init statement are re-used in each iteration.

带有 ForClause 的“ for”语句也受其条件控制，但是它还可以指定 init 和 post 语句，如赋值、递增或递减语句。Init 语句可以是简短的变量声明，但 post 语句不可以是段变量声明语句。Init 语句声明的变量在每次迭代中重用。【effecitve-go的Concurrency提过不想在block里面 使用的变量是重用的，区别于在post中的重用】

```
ForClause = [ InitStmt ] ";" [ Condition ] ";" [ PostStmt ] .
InitStmt = SimpleStmt .
PostStmt = SimpleStmt .
for i := 0; i < 10; i++ {
	f(i)
}
```

If non-empty, the init statement is executed once before evaluating the condition for the first iteration; the post statement is executed after each execution of the block (and only if the block was executed). Any element of the ForClause may be empty but the [semicolons](https://go.dev/ref/spec#Semicolons) are required unless there is only a condition. If the condition is absent, it is equivalent to the boolean value `true`.

如果非空，init 语句将在计算第一次迭代的条件之前执行一次; post 语句将在每次执行代码块之后执行(并且只在执行代码块时执行)。ForClause 中的任何元素都可以是空的，但是分号是`必需的`，除非只有一个条件。如果条件不存在，则等效于布尔值 true。

```
for cond { S() }    is the same as    for ; cond ; { S() }
for      { S() }    is the same as    for true     { S() }
```

### 9.3For statements with `range` clause带 range 子句

A "for" statement with a "range" clause iterates through all entries of an array, slice, string or map, or values received on a channel. For each entry it assigns *iteration values* to corresponding *iteration variables* if present and then executes the block.

带有“ range”子句的“ for”语句迭代接收的数组、片、字符串或映射或通道的值的所有条目。对于每个条目，它为相应的迭代变量赋值迭代值，如果存在，然后执行该块。

```
RangeClause = [ ExpressionList "=" | IdentifierList ":=" ] "range" Expression .

```

The expression on the right in the "range" clause is called the *range expression*, which may be an array, pointer to an array, slice, string, map, or channel permitting [receive operations](https://go.dev/ref/spec#Receive_operator).“ range”子句右边的表达式称为 range expression，它可以是数组、指向数组的指针、 slice、 string、 map 或允许接收操作的 channel【意思这个我们可以从这个通道获取值  for range <-channel】。

#### 疑问

 As with an assignment, if present the operands on the left must be [addressable](https://go.dev/ref/spec#Address_operators) or map index expressions; they denote the iteration variables. 与赋值一样，如果左边的操作数是可寻址的或映射索引表达式; 它们表示迭代变量。



If the range expression is a channel, at most one iteration variable is permitted, otherwise there may be up to two. If the last iteration variable is the [blank identifier](https://go.dev/ref/spec#Blank_identifier), the range clause is equivalent to the same clause without that identifier.

如果range expr是通道，则最多允许一个迭代变量，否则最多可能有两个。如果最后一个迭代变量是空标识符，那么 range 子句等效于没有该标识符的同一个子句。

The range expression `x` is evaluated once before beginning the loop, with one exception: if at most one iteration variable is present and `len(x)` is [constant](https://go.dev/ref/spec#Length_and_capacity), the range expression is not evaluated.

范围表达式 x 在开始循环之前计算一次，但有一个例外: 如果最多只有一个迭代变量，且 len (x)为常数，则不计算range expr。

Function calls on the left are evaluated once per iteration. For each iteration, iteration values are produced as follows if the respective iteration variables are present:

左边的函数调用每次迭代计算一次。对于每个迭代，如果存在相应的迭代变量，迭代值将产生如下:

```
Range expression                          1st value          2nd value

array or slice  a  [n]E, *[n]E, or []E    index    i  int    a[i]       E
string          s  string type            index    i  int    see below  rune
map             m  map[K]V                key      k  K      m[k]       V
channel         c  chan E, <-chan E       element  e  E
                channel的部分 接收channel，待确定
```

1. For an array, pointer to array, or slice value `a`, the index iteration values are produced in increasing order, starting at element index 0. If at most one iteration variable is present, the range loop produces iteration values from 0 up to `len(a)-1` and does not index into the array or slice itself. For a `nil` slice, the number of iterations is 0.

   对于数组、指向数组的指针或片值 a，索引迭代值按递增顺序生成，从元素索引0开始。如果最多只有一个迭代变量，那么范围循环将产生从0到 len (a)-1的迭代值，并且不会索引到数组中或者切片本身。对于 nil 切片，迭代次数为0。

   #### 为什么不索引自身

2. For a string value, the "range" clause iterates over the Unicode code points in the string starting at `byte` index 0. On successive iterations, the index value will be the index of the first byte of successive UTF-8-encoded code points in the string, and the second value, of type `rune`, will be the value of the corresponding code point. If the iteration encounters an invalid UTF-8 sequence, the second value will be `0xFFFD`, the Unicode replacement character, and the next iteration will advance a single byte in the string.

   对于字符串值，“ range”子句在 Unicode 代码上迭代，从字节索引0开始。在连续的迭代中，索引值将是字符串中连续 utf-8编码的代码点的第一个字节的索引，第二个值(rune 类型)将是对应的代码点的值。如果迭代遇到无效的 utf-8序列，第二个值将是0xfffd，即 Unicode 替换字符，下一次迭代将在字符串中前进一个字节。

3. The iteration order over maps is not specified and is not guaranteed to be the same from one iteration to the next. If a map entry that has not yet been reached is removed during iteration, the corresponding iteration value will not be produced. If a map entry is created during iteration, that entry may be produced during the iteration or may be skipped.

   #### 也就是map是无固定顺序的

    The choice may vary for each entry created and from one iteration to the next. If the map is `nil`, the number of iterations is 0.

   没有指定映射上的迭代顺序，也不能保证从一个迭代到下一个迭代是相同的。如果在迭代过程中删除了尚未到达的映射条目，则不会生成相应的迭代值。如果在迭代期间创建了映射条目，那么可以在迭代期间生成该条目，或者跳过该条目。对于创建的每个条目以及每次迭代的选择可能会有所不同。如果映射为 nil，迭代次数为0。

4. For channels, the iteration values produced are the successive values `sent on` the channel until the channel is [closed](https://go.dev/ref/spec#Close). If the channel is `nil`, the range expression blocks forever.

   #### 是发送给通道还是发送出去

   对于通道，生成的迭代值是在通道关闭之前在通道上发送的连续值。如果通道为 nil，则范围表达式将永远阻塞。

The iteration values are assigned to the respective iteration variables as in an [assignment statement](https://go.dev/ref/spec#Assignments).

迭代值分配给相应的迭代变量，如赋值语句中所示。

The iteration variables may be declared by the "range" clause using a form of [short variable declaration](https://go.dev/ref/spec#Short_variable_declarations) (`:=`). In this case their types are set to the types of the respective iteration values and their [scope](https://go.dev/ref/spec#Declarations_and_scope) is the block of the "for" statement; they are re-used in each iteration. If the iteration variables are declared outside the "for" statement, after execution their values will be those of the last iteration.

迭代变量可以由“ range”子句使用短变量声明(: =)的形式声明。在这种情况下，它们的类型被设置为各自迭代值的类型，它们的范围是“ for”语句的块; 它们在每次迭代中被重用。如果迭代变量在“ for”语句之外声明，那么在执行之后，它们的值将是最后一次迭代的值。

```
var testdata *struct {
	a *[7]int
}
for i, _ := range testdata.a {
	// testdata.a is never evaluated; len(testdata.a) is constant
	// i ranges from 0 to 6
	f(i)
}

var a [10]string
for i, s := range a {
	// type of i is int
	// type of s is string
	// s == a[i]
	g(i, s)
}

var key string
var val interface{}  // element type of m is assignable to val
m := map[string]int{"mon":0, "tue":1, "wed":2, "thu":3, "fri":4, "sat":5, "sun":6}
for key, val = range m {
	h(key, val)
}
// key == last map key encountered in iteration
// val == map[key]

var ch chan Work = producer()
for w := range ch {
	doWork(w)
}

// empty a channel
for range ch {}
```

## 10Go statements开启协程

A "go" statement starts the execution of a function call as an independent concurrent thread of control, or *goroutine*, within the same address space.

“ go”语句       开启一个函数执行，作为在相同的地址空间的 独立的并发控制线程或独立协程 执行

```
GoStmt = "go" Expression .
```

The expression must be a function or method call; it cannot be parenthesized. Calls of built-in functions are restricted as for [expression statements](https://go.dev/ref/spec#Expression_statements).

表达式必须是函数或方法调用; 它不能被括号括起来。内置函数的调用与表达式语句一样受到限制。

#### 协程开启后，并不需要里面的函数执行完毕代码才能继续往下走。

##### 同以前的认识有区别，比如必须函数或方法执行完下面的语句才可以继续执行。协程就是可一个独立空间时间让函数执行，但是暂时不理会走下面的代码，当协程空间里函数执行完毕协程才算终止。

The function value and parameters are [evaluated as usual](https://go.dev/ref/spec#Calls) in the calling goroutine, but unlike with a regular call, program execution does not wait for the invoked function to complete. Instead, the function begins executing independently in a new goroutine. When the function terminates, its goroutine also terminates. If the function has any return values, they are discarded when the function completes.【丢弃变量 是不是节省内存消耗了】

在调用 goroutine 中·函数值和参数的·计算和平常一样，但与常规调用不同的是，程序执行不等待被调用函数完成。相反，该函数开始在新 goroutine 中独立执行。当函数终止时，它的 goroutine 也终止。如果函数具有任何返回值，则在函数完成时丢弃这些返回值。

```
go Server() 
go func(ch chan<- bool) { for { sleep(10); ch <- true }} (c)
```

## 11Send statements发送声明

A send statement sends a value on a channel. The channel expression must be of [channel type](https://go.dev/ref/spec#Channel_types), the channel direction must permit `send operations`, and the type of the value to be sent must be [assignable](https://go.dev/ref/spec#Assignability) to the channel's element type.

Send 语句发送值给通道参考下面的ch<-3翻译为send to。通道表达式必须是通道类型，通道方向必须允许发送操作，要发送的值的类型必须可分配给通道的元素类型。

```
SendStmt = Channel "<-" Expression .   
Channel  = Expression . 表达式可以是identifier
```

Both the channel and the value expression are evaluated before communication begins. Communication blocks until the send can proceed.通道表达式和值表达式在通信开始之前被计算。通信会被阻塞知道 发送操作可以进行

#### 疑问：unbuffered和nil对于channel类型来讲不是一个意思吗？

​         错 make之后管道就不是Nil。make之后的管道可以是unbuffered也可以是buffered

#### 疑问:receiver怎样可以变成ready的状态，一个Unbuffered

 A send on an unbuffered channel can proceed if a receiver is ready.

 A send on a buffered channel can proceed if there is room in the buffer. 发送值给一个Buffer化的管道是可以的只要Buffer还有空间

A send on a closed channel proceeds by causing a [run-time panic](https://go.dev/ref/spec#Run_time_panics). 发送给关闭的channel会造成运行时错误

A send on a `nil` channel blocks forever.发送给Nil的管道会永远阻塞

```
ch <- 3  // send value 3 to channel ch
ch是receiver
```

## 12Select statements选择语句:

### select作用：供通信使用的类switch语句；

#### 选择a set of s|r op  ，会选择多个满足的case。

A "select" statement chooses which of a set of possible [send](https://go.dev/ref/spec#Send_statements) or [receive](https://go.dev/ref/spec#Receive_operator) operations will proceed. It looks similar to a ["switch"](https://go.dev/ref/spec#Switch_statements) statement but with the cases all referring to communication operations.“ select”语句选择一组可能的发送或接收操作中的哪一个将继续进行。它看起来类似于“ switch”语句，但cases都涉及到通信操作。

```
SelectStmt = "select" "{" { CommClause } "}" .
CommClause = CommCase ":" StatementList .
CommCase   = "case" ( SendStmt | RecvStmt ) | "default" . 
                      SemdStmt之前提过了
                                 RecvStmt   = [ ExpressionList "=" | IdentifierList ":=" ] RecvExpr .
                                 RecvExpr   = Expression .
```

##### 使用RecvStmt的case

A case with a RecvStmt may assign the result of a RecvExpr to one or two variables, which may be declared using a [short variable declaration](https://go.dev/ref/spec#Short_variable_declarations). The RecvExpr must be a (possibly parenthesized) `receive operation`. There can be at most one default case and it may appear anywhere in the list of cases.

使用 RecvStmt 的case可以将 RecvExpr 的结果分配给一个或两个变量，这些变量可以使用短变量声明声明。RecvExpr 必须是一个(可能是括号中的)接收操作。最多可以有一个默认情况，`它可能出现`case列表的任何地方。

#### select执行步骤

Execution of a "select" statement proceeds in several steps: “ select”语句的执行分为几个步骤:

##### 1{在进入select之前  }           RecvStmt[<- chan]  计算 chan这个部分,也就是；对于SendStmt【chan<-  expr】，计算 chan和expr.所谓side effects，在RecvStm体现为assignment赋值；RecvStmt的 ExpressionList  | IdentifierList未计算

##### 2{进入select}，根据pseudo-random selection只选择一个进行通信

##### 3通过2选择proceed的case;如果选择缺省case，不进行op，否则execution op

##### 4同点1不冲突：在点2的情况下，随机选择的case是RecvStmt，那么RecvStmt的 ExpressionList  | IdentifierList计算,这里的计算就是赋值操作；

1. For all the cases in the statement, the channel `operands` of receive operations and `the channel`and right-hand-side expressions of `send statements` are evaluated exactly once, in source order, `upon` entering the "select" statement. The result is a set of channels to receive from or send to, and the corresponding values to send. Any side effects in that evaluation will occur irrespective of which (if any) communication operation is selected to proceed. 对于语句中的所有情况，在进入“ select”语句时，接收操作的通道操作数以及发送语句的通道和`send statements`的右侧表达式【sendexpr】将按源顺序精确计算一次。结果是一组要从中接收或发送的通道，以及相应的要发送的值。无论选择哪个(如果有的话)通信操作进行，该计算中的任何副作用【】都将发生。

   Expressions on the left-hand side of a RecvStmt with a short variable declaration or assignment are not yet evaluated.具有短变量声明或赋值的 RecvStmt 左侧的表达式尚未计算。

2. If one or more of the communications can proceed, a single one that can proceed is `chosen` via a uniform pseudo-random selection. Otherwise, if there is a default case, that case is chosen. If there is no default case, the "select" statement blocks until at least one of the communications can proceed.如果可以进行一个或多个通信，则通过均匀的伪随机选择选择可以进行的单个通信。否则，如果存在缺省情况，则选择该情况。如果没有默认情况，“ select”语句将阻塞，直到至少有一个通信可以继续。

3. Unless the selected case is the default case, the respective communication operation is executed.

   除非所选的`case`是缺省`case`，否则将执行相应的通信操作。

4. If the selected case is a RecvStmt with a short variable declaration or an assignment, the left-hand side expressions are evaluated and the received value (or values) are assigned.

   如果选中的`case`是一个带有短变量声明或赋值的 RecvStmt，则计算左侧表达式并赋值接收到的值(或值)。

5. The statement list of the selected case is executed.

   执行所选案例的`StatementList`。
   
   ### 疑问var声明的 chan是不是nil channel：
   
   ##### 根据10DeclScope中var 下面的var c chan int 得到的是nil chan没有初始化

Since communication on `nil` channels can never proceed, a select with only `nil` channels and no default case blocks forever.

由于nil通道上的通信永远无法进行，只有nil的channel和不带缺省case的select语句 是永远阻塞的

```
var a []int
var c, c1, c2, c3, c4 chan int
var i1, i2 int
select {
接收 赋值
case i1 = <-c1:  
	print("received ", i1, " from c1\n")
发送
case c2 <- i2:
	print("sent ", i2, " to c2\n")
接收：短变量
case i3, ok := (<-c3):  // same as: i3, ok := <-c3                                
	if ok {
		print("received ", i3, " from c3\n")
	} else {
		print("c3 is closed\n")
	}
case a[f()] = <-c4:
	// same as:
	// case t := <-c4
	//	a[f()] = t
default:
	print("no communication\n")
}

for {  // send random sequence of bits to c   for循环
	select {
	case c <- 0:  // note: no statement, no fallthrough, no folding of cases
	case c <- 1:
	}
}

select {}  // block forever
```

## 13Return statements返回语句

A "return" statement in a function `F` terminates the execution of `F`, and optionally provides one or more result values. 

### defer比return优先执行完，return是用来终止函数的。

这个caller？

Any functions [deferred](https://go.dev/ref/spec#Defer_statements) by `F` are executed before `F` returns to its caller.

函数 f 中的“ return”语句终止 f 的执行，并可选地提供一个或多个结果值。F 中延迟的任何函数在 f 返回给它的调用者之前都会被执行。

```
ReturnStmt = "return" [ ExpressionList ] .
```

In a function without a result type, a "return" statement must not specify any result values.在没有结果类型的函数中，“ return”语句不能指定任何结果值。

```
func noResult() {
	return
}
```

There are three ways to return values from a function with a result type:有三种方法可以从结果类型的函数返回值:

1. The return value or values may be explicitly listed in the "return" statement. Each expression must be single-valued andassignable  to the corresponding element of the function's result type.

   返回值可以在“ return”语句中显式列出。每个表达式必须是单值和可分配的 到函数结果类型的对应元素

   ```
   func simpleF() int {
   	return 2
   }
   

func complexF1() (re float64, im float64) {
   	return -7.0, -4.0
}

2. The expression list in the "return" statement may be a single call to a multi-valued function. The effect is as if each value returned from that function were assigned to a temporary variable with the type of the respective value, followed by a "return" statement listing these variables, at which point the rules of the previous case apply.

   #### 没理解清楚 返回的是... 比较细节暂时放下

    “ return”语句中的表达式列表可以是对多值函数的单个调用。其效果就好像是将该函数返回的每个值分配给一个临时变量，临时变量带着each value值的类型，后面跟着一个列出这些变量的“ return”语句，此时适用前一种情况的规则

   func complexF2() (re float64, im float64) {
   	return complexF1()
   }

3. The expression list may be empty if the function's result type specifies names for its result parameters 

    如果函数的结果类型为其指定了结果参数名称，则表达式列表可能为空

   . The result parameters act as ordinary local variables and the function may assign values to them as necessary. The "return" statement returns the values of these variables.结果参数作为普通的局部变量，函数可以根据需要为它们赋值。返回语句返回这些变量的值

   func complexF3() (re float64, im float64) {
   	re = 7.0
   	im = 4.0
   	return
   }

   func (devnull) Write(p []byte) (n int, _ error) {
   	n = len(p)
   	return
   }

#### 同defer比return先执行有冲突吗？不冲突

Regardless of how they are declared, all the result values are initialized to the [zero values](https://go.dev/ref/spec#The_zero_value) for their type upon entry to the function. A "return" statement that specifies results sets the result parameters before any deferred functions are executed.

无论如何声明它们，在进入函数时，所有结果值都被初始化为其类型的零值。在被defer的函数执行之前【上面提defer比return先执行】，return语句早早就指定 哪些参数 是作为结果返回。比如函数内声明多个变量a,b,c我先指定我返回a作为结果参数。就这意思。

defer比return先执行，defer执行之前return是声明了返回的变量。

Implementation restriction: A compiler may disallow an empty expression list in a "return" statement if a different entity (constant, type, or variable) with the same name as a result parameter is in [scope](https://go.dev/ref/spec#Declarations_and_scope) at the place of the return.

实现限制: 如果返回位置的范围中有与结果参数同名的不同实体(常量、类型或变量) ，则编译器可能禁止“ return”语句中的空表达式列表。

func f(n int) (res int, err error) {
	if _, err := f(n-1); err != nil {
		return  // invalid return statement: err is shadowed
	}
	return
}
```

## 14Break statements

### break结束最里面的一整个loop执行

A "break" statement terminates execution of the innermost ["for"](https://go.dev/ref/spec#For_statements), ["switch"](https://go.dev/ref/spec#Switch_statements), or ["select"](https://go.dev/ref/spec#Select_statements) statement within the same function.

“ break”语句终止同一函数中最里面的“ for”、“ switch”或“ select”语句的执行。

```
BreakStmt = "break" [ Label ] .
```

#### 特殊情况:加上label

If there is a label, it must be that of an enclosing "for", "switch", or "select" statement, and that is the one whose execution terminates.

如果有一个标签，它必须是一个封闭的“ for”、“ switch”或“ select”语句的标签，而这个标签的执行将终止。

```
OuterLoop:
	for i = 0; i < n; i++ {
		for j = 0; j < m; j++ {
			switch a[i][j] {
			case nil:
				state = Error
				break OuterLoop
			case item:
				state = Found
				break OuterLoop
			}
		}
	}
```

## 15Continue statements

### continue是开启内层loop的下一次迭代而不是结束整个内层loop

A "continue" statement begins the next iteration of the innermost ["for" loop](https://go.dev/ref/spec#For_statements) at its post statement. The "for" loop must be within the same function.

“ continue”语句开始其 post 语句中最内层的“ for”循环的下一次迭代。“ for”循环必须位于同一函数内。

```
ContinueStmt = "continue" [ Label ] .
```

#### 特殊情况:加个label，执行label

If there is a label, it must be that of an enclosing "for" statement, and that is the one whose execution advances.

如果有一个标签，那么它必须是一个封闭的“ for”语句，而这个语句的执行将会进行。

```
RowLoop:
	for y, row := range rows {
		for x, data := range row {
			if data == endOfRow {
				continue RowLoop
			}
			row[x] = data + bias(x, y)
		}
	}
```

## 16Goto statements:跳转到带Label语句

A "goto" statement transfers control to the statement with the corresponding label within the same function.

“ goto”语句将控制流程转移到同一函数中具有相应标签的语句。

```
GotoStmt = "goto" Label .
goto Error

### 使用Goto有要求

#### 1不可以造成 变量声明遗失：具体意思有点疑惑

Executing the "goto" statement must not cause any variables to come into [scope](https://go.dev/ref/spec#Declarations_and_scope) that were not already in scope at the point of the goto. For instance, this example:

	goto L  // BAD
	v := 3
L:


is erroneous because the jump to label `L` skips the creation of `v`.是错误的，因为跳转到标签 l 会跳过 v 的创建。

#### 2不可以跳转到块内

A "goto" statement outside a [block](https://go.dev/ref/spec#Blocks) cannot jump to a label inside that block. For instance, this example:

块外的“ goto”语句不能跳转到块内的标签。例如:

if n%2 == 1 {
	goto L1
}
for n > 0 {
	f()
	n--
L1:
	f()
	n--
}


is erroneous because the label `L1` is inside the "for" statement's block but the `goto` is not.

是错误的，因为标签 l1位于“ for”语句的块中，而 goto 则不是。

### 17Fallthrough statements：没有体会fallthrough

### 参考switch stmt，有疑问

A "fallthrough" statement transfers control to the first statement of the next case clause in an [expression "switch" statement](https://go.dev/ref/spec#Expression_switches). It may be used only as the final non-empty statement in such a clause.

“ fallthrough”语句将控制转移到表达式“ switch”语句中下一个 case 子句的第一个语句。它只能用作这种子句中最后的非空语句。

FallthroughStmt = "fallthrough" .

## 18Defer statements推迟语句

A "defer" statement invokes a function whose execution is deferred to the moment the surrounding function returns, either because the surrounding function executed a [return statement](https://go.dev/ref/spec#Return_statements), reached the end of its [function body](https://go.dev/ref/spec#Function_declarations), or because the corresponding goroutine is [panicking](https://go.dev/ref/spec#Handling_panics).

一个“推迟”语句调用一个函数，该函数的执行被推迟到周围函数返回的那一刻，这要么是因为周围函数执行了一个 return 语句，到达了函数体的末尾，要么是因为相应的 goroutine 感到恐慌。

DeferStmt = "defer" Expression .

### defer的是方法或函数

The expression must be a function or method call; it cannot be parenthesized. Calls of built-in functions are restricted as for [expression statements](https://go.dev/ref/spec#Expression_statements).表达式必须是函数或方法调用; 它不能被括号括起来。内置函数的调用与表达式语句一样受到限制。

### defer的函数参数先计算好，但是在外层函数返回的前一瞬才执行

Each time a "defer" statement executes, the function value and parameters to the call are [evaluated as usual](https://go.dev/ref/spec#Calls) and saved anew but the actual function is `not invoked.` Instead, deferred functions are invoked immediately before the surrounding function returns, in the `reverse` order they were deferred. That is, if the surrounding function returns through an explicit [return statement](https://go.dev/ref/spec#Return_statements), deferred functions are executed *after* any result parameters are set by that return statement but *before* the function returns to its caller. If a deferred function value evaluates to `nil`, execution [panics](https://go.dev/ref/spec#Handling_panics) when the function is invoked, not when the "defer" statement is executed.

每次执行“ defer”语句时，调用的函数值和参数都会像通常一样进行计算并重新保存，但不会调用实际的函数。相反，延迟函数在周围函数返回之前被调用，调用顺序与延迟的顺序相反。也就是说，如果周围的函数通过显式的 return 语句返回，那么在 return 语句设置任何结果参数之后，在函数返回给调用者之前，将执行延迟函数。如果延迟函数值的计算结果为 nil，则在调用函数时执行会感到恐慌，而不是在执行“ defer”语句时。

For instance, if the deferred function is a [function literal](https://go.dev/ref/spec#Function_literals) and the surrounding function has [named result parameters](https://go.dev/ref/spec#Function_types) that are in scope within the literal, the deferred function may access and modify the result parameters before they are returned. If the deferred function has any return values, they are discarded when the function completes. (See also the section on [handling panics](https://go.dev/ref/spec#Handling_panics).)

例如，如果延迟函数是一个匿名函数字面量，周围函数是带有命名的结果参数，参数范围是包含在匿名defer函数 字面量内，那么延迟函数可以在返回结果参数之前访问和修改结果参数。如果延迟函数有任何返回值，则在函数完成时将丢弃这些返回值。(请参阅处理恐慌一节)

lock(l)
defer unlock(l)  // unlocking happens before surrounding function returns

// prints 3 2 1 0 before surrounding function returns
for i := 0; i <= 3; i++ {
	defer fmt.Print(i)
}

// f returns 42
func f() (result int) {
	defer func() {
		// result is accessed after it was set to 6 by the return statement
		result *= 7
	}()
	return 6//之前提过defer比return先执行但是在defer执行之前,return就把result parameter也就是6设置给result:resultl
}