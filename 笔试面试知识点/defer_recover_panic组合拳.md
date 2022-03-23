---
desir1822
23.3.22
讲讲异常处理
---



[TOC]

# （1）首先参考一篇blog 

[参考blog]: https://go.dev/blog/defer-panic-and-recover

## Defer, Panic, and Recover

Andrew Gerrand
4 August 2010



### 1.1以下讲了什么:defer的优势

不用我们去记那一步应该文件关闭，会自动执行，关键端黑体加粗部分。

Go has the usual mechanisms for control flow: if, for, switch, goto. It also has the go statement to run code in a separate goroutine. Here I’d like to discuss some of the less common ones: defer, panic, and recover.

A **defer statement** pushes a function call onto a list. The list of saved calls is executed after the surrounding function returns. Defer is commonly used to simplify functions that perform various clean-up actions.

For example, let’s look at a function that opens two files and copies the contents of one file to the other:

```
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

This works, but there is a bug. If the call to os.Create fails, the function will return without closing the source file. This can be easily remedied by putting a call to src.Close before the second return statement, but if the function were more complex the problem might not be so easily noticed and resolved. By introducing defer statements we can ensure that the files are always closed:

```
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

**Defer statements allow us to think about closing each file right after opening it, guaranteeing that, regardless of the number of return statements in the function, the files *will* be closed.**

### 1.2defer  3条规则

#### 1.2.1 defer语句一出现，defer使用的变量就已经计算好之后的·修改不生效。

The behavior of defer statements is straightforward and predictable. There are three simple rules:

1. *A deferred function’s arguments are evaluated when the defer statement is evaluated.*

In this example, the expression “i” is evaluated when the Println call is deferred. The deferred call will print “0” after the function returns.

```
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}

```

#### 1.2.2 defer先进后出 栈的思想

*Deferred function calls are executed in Last In First Out order after the surrounding function returns.*

This function prints “3210”:

```
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)//这一条可以参考第一条 已经确定
    }
}
```

#### 1.2.3    defer对返回值有影响

[结果参数]: https://go.dev/doc/effective_go#named-results

[^Named result parameters命名结果参数]: The return or result "parameters" of a Go function can be given names and used as regular variables, just like the incoming parameters. When named, they are initialized to the zero values for their types when the function begins; if the function executes a `return` statement with no arguments, the current values of the result parameters are used as the returned values.Go 函数的返回或结果“参数”可以指定名称并用作常规变量，就像传入的参数一样。当命名时，当函数开始时，它们被初始化为它们类型的零值; 如果函数执行一个没有参数的返回语句，则结果参数的当前值被用作返回值。

大概就是说,结果参数函数刚开始是对应类型zero-value,之后

[第一种方式]: https://go.dev/ref/spec#Return_statements

[^]: There are three ways to return values from a function with a result type:有三种方法可以从结果类型的函数返回值:The return value or values may be explicitly listed in the "return" statement. Each expression must be single-valued and 返回值可以在“ return”语句中显式列出。每个表达式必须是单值和[assignable 可分配的](https://go.dev/ref/spec#Assignability) to the corresponding element of the function's result type. 到函数结果类型的对应元素`func simpleF() int { return 2 } `

*Deferred functions may read and assign to the returning function’s named return values.*

*延迟函数可以读取并赋值给返回函数的命名返回值。*

In this example, a deferred function increments the return value i *after* the surrounding function returns. Thus, this function returns 2:

在此示例中，延迟函数 在周围函数返回*后递增返回值 i。*因此，此函数返回 2：

```
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```

##### 上述的影响函数开始时  i时int 0值 0，然后return 1将 1赋值给i  之后  defer又对i自增  最终 返回2.



### 1.3  结合Panic 和Recover



#### 1.3.1重点：Panic作用，停止流程控制，Panic之后的语句不执行

**`Panic` is a built-in function that stops the ordinary flow of control and begins *panicking*.** When the function F calls panic, execution of F stops, any deferred functions in F are executed normally, and then F returns to its caller. To the caller, F then behaves like a call to panic. **The process continues up the stack until all functions in the current goroutine have returned,**（已经入栈的defer函数会按照LIFO方式返回，panic之后的defer由于程序停止时不入栈的） at which point the program crashes（defer都执行完毕后，程序正式崩溃，之后的步骤就不在进行）. Panics can be initiated by invoking panic directly. They can also be caused by runtime errors, such as out-of-bounds array accesses.

**Panic**是一个内置函数，可以停止普通的控制流程并开始*恐慌*。当函数 F 调用 panic 时，F 的执行停止，F 中的所有延迟函数都正常执行，然后 F 返回其调用者。对调用者来说，F 的行为就像是对恐慌的调用。此时程序崩溃。恐慌可以通过直接调用恐慌来启动。它们也可能是由运行时错误引起的，例如越界数组访问。

**`Recover` is a built-in function that regains control of a panicking goroutine.** Recover is only useful inside deferred functions. During normal execution, a call to recover will return nil and have no other effect. If the current goroutine is panicking, a call to recover will capture the value given to panic and resume normal execution.

#### 1.3.2recover如果单纯panic程序是执行defer后崩溃，不执行后续

##### 注意recover返回的值是panic的调用参数

Here’s an example program that demonstrates the mechanics of panic and defer:

```
package main

import "fmt"

func main() {
    f()//由于f()发生panic  下面的打印语句不执行
    fmt.Println("Returned normally from f.")//如果f()执行了recover() 重获goroutine这条语句可以执行
}

func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)//如果有recover会打印两部分一部分是字符串 一部分是r，r是panic的调用参数i的值4
        }
    }()
    fmt.Println("Calling g.")
    g(0)//发生panic 后续不会执行打印
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}
```

The function g takes the int i, and panics if i is greater than 3, or else it calls itself with the argument i+1. The function f defers a function that calls recover and prints the recovered value (if it is non-nil). Try to picture what the output of this program might be before reading on.

The program will output:

```
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
Recovered in f 4
Returned normally from f.
```

##### 删除recover,程序terminate

f()函数中fmt.Println("Returned normally from g.")不执行，main()中最后一条语句也不执行 fmt.Println("Returned normally from f.")

注意：一个是说正常退出

If we remove the deferred function from f the panic is not recovered and reaches the top of the goroutine’s call stack, terminating the program. This modified program will output:

```
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
panic: 4

panic PC=0x2a9cd8
[stack trace omitted]
```

### 1.4 panic ,recover, defer其他妙用（可不看）

For a real-world example of **panic** and **recover**, see the [json package](https://go.dev/pkg/encoding/json/) from the Go standard library. It encodes an interface with a set of recursive functions. If an error occurs when traversing the value, panic is called to unwind the stack to the top-level function call, which recovers from the panic and returns an appropriate error value (see the ‘error’ and ‘marshal’ methods of the encodeState type in [encode.go](https://go.dev/src/pkg/encoding/json/encode.go)).

The convention in the Go libraries is that even when a package uses panic internally, its external API still presents explicit error return values.

Other uses of **defer** (beyond the file.Close example given earlier) include releasing a mutex:

```
mu.Lock()
defer mu.Unlock()
```

printing a footer:

```
printHeader()
defer printFooter()
```

and more.

In summary, the defer statement (with or without panic and recover) provides an unusual and powerful mechanism for control flow. It can be used to model a number of features implemented by special-purpose structures in other programming languages. Try it out.

**Next article:** [Go Wins 2010 Bossie Award](https://go.dev/blog/bossie)
**Previous article:** [Share Memory By Communicating](https://go.dev/blog/codelab-share)
**[Blog Index](https://go.dev/blog/all)**

# （2）3巨头分别介绍

## 2.1Defer

Go's `defer` statement schedules a function call (the *deferred* function) to be run immediately before the function executing the `defer` returns. It's an unusual but effective way to deal with situations such as resources that must be released regardless of which path a function takes to return. The canonical examples are unlocking a mutex or closing a file.

Go 的 defer 语句调度一个函数调用(延迟函数) ，在函数执行 defer 返回之前运行。这是一种不同寻常但有效的方法来处理这样的情况，比如无论函数采用哪种返回路径，都必须释放资源。规范的示例是解锁互斥对象或关闭文件。

```
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
//return之前先执行defer
```

Deferring a call to a function such as `Close` has two advantages. First, it guarantees that you will never forget to close the file, a mistake that's easy to make if you later edit the function to add a new return path. Second, it means that the close sits near the open, which is much clearer than placing it at the end of the function.

延迟对 Close 等函数的调用有两个好处。首先，它保证您永远不会忘记关闭文件，如果稍后编辑该函数以添加新的返回路径，那么很容易出错。其次，这意味着 close 位于打开位置附近，这比将它放在函数的末尾要清楚得多。

The arguments to the deferred function (which include the receiver if the function is a method) are evaluated when the *defer* executes, not when the *call* executes. Besides avoiding worries about variables changing values as the function executes, this means that a single deferred call site can defer multiple function executions. Here's a silly example.

延迟函数的参数(如果函数是方法，则包括接收者)在延迟执行时计算，而不是在调用执行时计算。除了避免担心变量在函数执行时改变值之外，这意味着单个延迟调用站点可以推迟多个函数的执行。这里有一个愚蠢的例子。

```
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
打印 4 3 2 1 0 
```

Deferred functions are executed in LIFO order, so this code will cause `4 3 2 1 0` to be printed when the function returns. A more plausible example is a simple way to trace function execution through the program. We could write a couple of simple tracing routines like this:

延迟函数是按照后进先出的顺序执行的，所以当函数返回时，这段代码将导致打印43210。一个更合理的例子是一种通过程序跟踪函数执行的简单方法。我们可以编写几个简单的跟踪例程，如下所示:

```
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

### 2.1.2重点：参考（1）博客内容第一条defer规则，变量在一出现defer语句就计算好了，在以下代码体现为



We can do better by exploiting the fact that arguments to deferred functions are evaluated when the `defer` executes. The tracing routine can set up the argument to the untracing routine. This example:

我们可以更好地利用延迟函数的参数在延迟执行时被求值这一事实。跟踪例程可以将参数设置为取消跟踪例程。这个例子:

```
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a")) 2-2
    fmt.Println("in a")   2-1
}

func b() {
    defer un(trace("b"))// 先调用trace,然后调用打印in b 然后调用 a()   最后defer un("b") 
    fmt.Println("in b") //                        调用 a()  先调用trace(a)然后打印in a 最后 defer un("a")
    a()                   
}
func main() {
    b()
}
```

prints

```
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

For programmers accustomed to block-level resource management from other languages, `defer` may seem peculiar, but its most interesting and powerful applications come precisely from the fact that it's not block-based but function-based. In the section on `panic` and `recover` we'll see another example of its possibilities.

C:\Users\001\go\go1.16\src\runtime\runtime2.go

defer内部实现是一个链表，链表元素类型是`_defer`结构体，其中的`link`字段指向下一个`_defer`地址，当定义一个defer语句时候，系统内部会将defer函数转换成_defer结构体，并放在链表头部，最后执行时候，系统会从链表头部开始依次执行，这也就是多个defer的执行顺序是First In Last out的原因。

## 2.2Panic没讲太多东西

### 讲了个panic让程序停止执行，panic传入参数可以是任意类型

The usual way to report an error to a caller is to return an `error` as an extra return value. The canonical `Read` method is a well-known instance; it returns a byte count and an `error`. But what if the error is unrecoverable? Sometimes the program simply cannot continue.

通常向调用方报告错误的方法是将错误作为额外的返回值返回。规范的 Read 方法是一个众所周知的实例; 它返回一个字节计数和一个错误。但是，如果错误是不可恢复的呢？有时程序根本无法继续。

For this purpose, there is a built-in function **`panic` that in effect creates a run-time error that will stop the program (but see the next section).** The function **take*s a single argument of arbitrary type***—often a string—to be printed as the program dies. It's also a way to indicate that something impossible has happened, such as exiting an infinite loop.

为此，有一个内置的函数恐慌，它实际上创建了一个将停止程序的运行时错误(但请参阅下一节)。该函数接受一个任意类型的参数(通常是一个字符串) ，在程序结束时打印出来。这也是一种表明某些不可能发生的事情已经发生的方式，例如退出一个无限循环。

```
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```

This is only an example but real library functions should avoid `panic`. If the problem can be masked or worked around, it's always better to let things continue to run rather than taking down the whole program. One possible counterexample is during initialization: if the library truly cannot set itself up, it might be reasonable to panic, so to speak.

这只是一个例子，但是真正的库函数应该避免恐慌。如果问题可以被掩盖或解决，那么最好让事情继续运行，而不是毁掉整个程序。一个可能的反例是在初始化过程中: 如果库确实不能设置自己，那么可以这么说，恐慌是合理的。

```
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

## 2.3Recover

### 2.3.1展开堆栈，里面是已入栈的defer函数

![image-20220323180849572](C:\Users\001\AppData\Roaming\Typora\typora-user-images\image-20220323180849572.png)

func.1后执行于func.2

### 2.3.2调用panic

When `panic` is called, including implicitly for run-time errors such as indexing a slice out of bounds or failing a type assertion, it immediately stops execution of the current function and begins unwinding the stack of the goroutine, running any deferred functions along the way. If that unwinding reaches **the top of the goroutine's stack**（**栈顶就是最后一个defer执行完毕**）, the program dies. However, it is possible to use the built-in function `recover` to regain control of the goroutine and resume normal execution.

当调用 panic 时，包括隐式地针对运行时错误(如索引一个片段超出界限或类型断言失败) ，它会立即停止当前函数的执行，并开始展开 goroutine 的堆栈，沿途运行任何延迟的函数。如果展开到 goroutine 堆栈的顶部，程序就会死亡。但是，可以使用内置函数 recover 来重新获得对 goroutine 的控制并恢复正常执行。

### 2.3.3调用recover

\A call to `recover` stops the unwinding and returns the argument passed to `panic`. Because the only code that runs while unwinding is inside deferred functions, `recover` is only useful inside deferred functions.调用 recover 会停止展开，并返回传递给 panic 的参数。因为唯一在展开时运行的代码在延迟函数中，所以 recover 只在延迟函数中有用。

#### 2.3.3.1停止展开？ 

#### 2.3.3.2recover返回传给panic的参数

![image-20220323190859847](C:\Users\001\AppData\Roaming\Typora\typora-user-images\image-20220323190859847.png)

One application of `recover` is to shut down a failing goroutine inside a server without killing the other executing goroutines.

Recover 的一个应用是关闭服务器内部的故障 goroutine，而不会杀死其他正在执行的 goroutine。

```
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

In this example, if `do(work)` panics, the result will be logged and the goroutine will exit cleanly without disturbing the others. There's no need to do anything else in the deferred closure; calling `recover` handles the condition completely.

在这个示例中，如果 do (work)感到恐慌，那么结果将被记录，goroutine 将干净地退出，而不会干扰其他人。在延迟闭包中不需要执行任何其他操作; 调用 recover 可以完全处理这种情况。

Because `recover` always returns `nil` unless called directly from a deferred function, deferred code can call library routines that themselves use `panic` and `recover` without failing. As an example, the deferred function in `safelyDo` might call a logging function before calling `recover`, and that logging code would run unaffected by the panicking state.

因为除非直接从延迟函数调用，否则 recover 总是返回 nil，所以延迟代码可以调用自己使用 panic 和 recover 而不会失败的库例程。例如，safelyDo 中的 deferred 函数可能在调用 recover 之前调用日志函数，而日志代码的运行不会受到恐慌状态的影响。

# （3）一篇论坛博文

[]: https://juejin.cn/post/6886710490530054158

#### --可以修改函数中的命名返回值--

#### --匿名返回值(指针||值类型)--

