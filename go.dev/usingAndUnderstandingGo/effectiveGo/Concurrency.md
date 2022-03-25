---
effecitve-go
2022.1.05
并发编程是一个很大的话题，这里只有一些Go特有的亮点。
阅读该文档最好先了解《操作系统》的进程管理，原理是类似的。
---



[TOC]



# Concurrency并发性

## 1Share by communicating

Concurrent programming in many environments is made difficult by the subtleties required to implement correct access to shared variables. Go encourages a different approach in which shared values are passed around on channels and, in fact, never actively shared by separate threads of execution. Only one goroutine has access to the value at any given time. Data races cannot occur, by design. To encourage this way of thinking we have reduced it to a slogan:

许多环境中的并发编程由于`subtleties`需要实现对共享变量的正确访问而变得困难。Go 鼓励一种不同的方法，在这种方法中，共享的值在通道中传递，实际上，不会被单独的执行线程主动共享。在任何给定时间只有一个 goroutine 可以访问该值。根据设计，数据竞争不能发生。为了鼓励这种思维方式，我们把它简化为一句口号:

> Do not communicate by sharing memory; instead, share memory by communicating.不要通过共享内存来进行通信，而是通过通信来共享内存。

This approach can be taken too far. Reference counts may be best done by putting a mutex around an integer variable, for instance. But as a high-level approach, using channels to control access makes it easier to write clear, correct programs.

这种方法意义深远。例如，引用计数通过为整数变量添加互斥锁来很好地实现。 但作为一种高级方法，通过信道来控制访问能够让你写出更简洁，正确的程序。

One way to think about this model is to consider a typical single-threaded program running on one CPU. It has no need for synchronization primitives. Now run another such instance; it too needs no synchronization. Now let those two communicate; if the communication is the synchronizer, there's still no need for other synchronization. Unix pipelines, for example, fit this model perfectly. Although Go's approach to concurrency originates in Hoare's Communicating Sequential Processes (CSP), it can also be seen as a type-safe generalization of Unix pipes.

我们可以从典型的单线程运行在单CPU之上的情形来审视这种模型。它无需提供同步原语。 现在考虑另一种情况，它也无需同步。现在让它们俩进行通信。若将通信过程看做同步着， 那就完全不需要其它同步了。例如，Unix管道就与这种模型完美契合。 尽管Go的并发处理方式来源于Hoare的通信顺序处理（CSP）， 它依然可以看做是类型安全的Unix管道的实现。

## 2Go_routines  Go协程

### 2.1协程定义

They're called *goroutines* because the existing terms—threads, coroutines, processes, and so on—convey inaccurate connotations. A goroutine has a simple model: it is a function executing concurrently with other goroutines in the same address space. It is lightweight, costing little more than the allocation of stack space. And the stacks start small, so they are cheap, and grow by allocating (and freeing) heap storage as required.

它们之所以被称为 goroutine，是因为现有的术语ーー线程、协同程序、进程等等ーー传达了不准确的内涵。Goroutine 有一个简单的模型: 它是与其它Go程并发运行在同一地址空间的函数。它是轻量级的， 所有小号几乎就只有栈空间的分配。而且栈最开始是非常小的，所以它们很廉价， 仅在需要时才会随着堆空间的分配（和释放）而变化。

Goroutines are multiplexed onto multiple OS threads so if one should block, such as while waiting for I/O, others continue to run. Their design hides many of the complexities of thread creation and management.

Go程在多线程操作系统上可实现多路复用，因此若一个线程阻塞，比如说等待I/O， 那么其它的线程就会运行。Go程的设计隐藏了线程创建和管理的诸多复杂性。

Goroutine Prefix a function or method call with the `go` keyword to run the call in a new goroutine. When the call completes, the goroutine exits, silently. (The effect is similar to the Unix shell's `&` notation for running a command in the background.)

在函数或方法前添加 `go` 关键字能够在新的Go程中调用它。当调用完成后， 该Go程也会安静地退出。（效果有点像Unix Shell中的 `&` 符号，它能让命令在后台运行。）

```
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

A function literal can be handy in a goroutine invocation.函数可以方便的进行 协程调用

```
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}必须调用函数
```

### 2.2闭包:需要进一步理解。大概就是不是一次次的重新声明变量。

In Go, function literals are `closures`: the implementation makes sure the variables referred to by the function survive as long as they are active.

[FuncLit地址]: https://github.com/desir1822/learingGoLan/tree/main/goSpecification

These examples aren't too practical because the functions have no way of signaling completion. For that, we need channels.

在Go中，函数字面都是闭包：其实现在保证了函数内引用变量的生命周期与函数的活动时间相同。

这些函数没什么实用性，因为它们没有实现完成时的信号处理。因此，我们需要信道。

## 3Channels信道

Like maps, channels are allocated with `make`, and the resulting value acts as a reference to an underlying data structure. If an optional integer parameter is provided, it sets the buffer size for the channel. The `default` is zero, for an unbuffered or synchronous channel.与映射一样，通道通过 `make` 分配，结果值作为对底层数据结构的引用。如果提供了一个可选的整数参数，它将设置通道的缓冲区大小。默认值是零，表示不带缓冲的或同步的信道。

```
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

Unbuffered channels combine communication—the exchange of a value—with synchronization—guaranteeing that two calculations (goroutines) are in a known state.

无缓冲信道在通信时会同步交换数据，它能确保（两个协程的）计算处于确定状态。[参考无缓冲信道交换数据那张图就明白啥意思了]

There are lots of nice idioms using channels. Here's one to get us started. In the previous section we launched a sort in the background. A channel can allow the launching goroutine to wait for the sort to complete.

使用通道的习惯用法很多。这里有一个可以让我们开始。在前面的部分中，我们在后台启动了一个排序。一个通道可以允许启动 goroutine 等待排序完成。

```
c := make(chan int)  // Allocate a channel.一个unbuffered的管道
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
协程的特点，不用等待协程函数执行完就会直接执行下面的代码，这里先执行doSomethingForWhile() 然后如果协程没有执行完毕 <-c是无法进行的【阻塞】，只有等排序完成 c先收到1 <-c才可以进行
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.等待完成，丢弃发送的值
```

### 3.1阻塞

#### 1what is block>>

正在运行的进程由于提出系统服务请求（如I/O操作），但因为某种原因未得到操作系统的立即响应，或者需要从其他合作进程获得的数据尚未到达等原因，该进程只能调用阻塞原语把自己阻塞，等待相应的事件出现后才被唤醒。

#### 2不阻塞意味着什么？

在代码中意味着这一行代码顺利执行，比如上面的c<-1执行完毕，协程也执行完成。

#### 3发生阻塞的情况

Receivers always block until there is data to receive.  接收器在 有数据可供接收之前一直都是封锁的；就是上面的最后一句<-c只有c之前接收过1 接收器才会进行，不阻塞 当然这里没有显式指出receiver是谁。

[未缓冲通道]: https://blog.csdn.net/zhengwish/article/details/103165180

If the channel is unbuffered, the sender blocks until the receiver has received the value. 如果通道没有buffered，发送方一直阻塞直到接收器 接收到值。c就是一个unbuffered的channel因为make时没有指定cap，然后协程里面 c<-1 会一直阻塞，阻塞就会导致<-c失败，一旦1发送成功给c就不阻塞<-c也就可以进行。需要提醒的是不需要等待协程执行完毕就可以跑下一步比如doSomethingForAWhile()

If the channel has a buffer, the sender blocks only until the value has been copied to the buffer;发送方一直阻塞直到值已经赋值给buffer了。

 if the buffer is full, this means waiting until some receiver has retrieved a value.buffer若缓冲区已满，发送者会一直等待直到某个接收者取出一个值为止。

### 3.2通道的用途：信号量

#### 3.2.1消耗资源的协程创建：先创建channel而后阻塞sem

A buffered channel can be used like a semaphore, for instance to limit throughput. In this example, incoming requests are passed to `handle`, which sends a value into the channel, processes the request, and then receives a value from the channel to ready the “semaphore” for the next consumer. The capacity of the channel buffer limits the number of `simultaneous` calls to `process`.

缓冲通道可以像信号量一样使用，例如限制吞吐量。在这个例子中，传入的请求被传递给 handle，它向通道发送一个值，处理请求，然后从通道接收一个值，以便让“信号量”准备迎接下一次请求。信道缓冲区的容量决定了同时调用 `process` 的数量上限，因此我们在初始化时首先要填充至它的容量上限。

```
var sem = make(chan int, MaxOutstanding)   一个buffer化的通道

func handle(r *Request) {
    sem <- 1    // Wait for active queue to drain.
    process(r)  // May take a long time.
    <-sem       // Done; enable next request to run.
}

func Serve(queue chan *Request) {//信道类型    接收或发送请求 
    for {
        req := <-queue       req接收请求队列
        go handle(req)  // Don't wait for handle to finish.   无需等待handle执行结束
    }
}
```

Once `MaxOutstanding` handlers are executing `process`, any more will block trying to send into the filled channel buffer, until one of the existing handlers finishes and receives from the buffer.一旦 MaxOutstanding个handle在执行`process(r)`，如果还有其他的handler往已满的缓冲发送值，将会阻塞，直到现有处理程序[MaxOutStanding其中之一]之一完成并从缓冲区接收，其他的handler才不会阻塞。

##### 注意点

[我们要注意的是`dont wait for handle to finsh`，也就是说 刚开始可能1发送多次到了sem sem直接达到MaxOutStanding，也就是sem这个通道的Buffer已满，其他的handle还想把1发送给sem是不可行的因为sem已满，只要前面的handles中有一个handle 执行了 <-sem也就是接收 sem buffer里面的1 也就是只要有一个handle是Done的状态 就不会封锁了。这不就是 `死锁`]

##### 疑问：为啥这种消耗资源

This design has a problem, though: `Serve` creates a new goroutine for every incoming request, even though only `MaxOutstanding` of them can run at any moment. As a result, the program can consume unlimited resources if the requests come in too fast.然而，它却有个设计问题：尽管只有 `MaxOutstanding` 个Go程能同时运行，但 `Serve` 还是为每个进入的请求都创建了新的Go程。其结果就是，若请求来得很快， 该程序就会无限地消耗资源。为了弥补这种不足，我们可以通过修改 `Serve` 来限制创建Go程，这是个明显的解决方案，但要当心我们修复后出现的Bug。

我们要注意 sem是局部变量 r是一个request。对于`Serve`来讲 传入参数 是一个双dir管道，值是req 也就是它可以接收发送 too much reqs

里面循环  一直在创建 goroutine ，即使 由于sem的buffer size限制，只有 MaxOutStanding个可以同时运行。go func()()一定创建协程 但是由于sem的限制，超出MaxOutStanding数量的其他协程 是出于Block阻塞状态。

##### go关键字只管创建协程而不管协程的状态。

#### 3.2.2改变消耗资源：先阻塞sem 从而限制创建

 We can address that deficiency by changing `Serve` to gate the creation of the goroutines. Here's an obvious solution, but beware it has a bug we'll fix subsequently:我们可以通过改变`Serve`取消创建too much goroutines 来解决这个不足。这里有一个显而易见的解决方案，但是要注意它有一个 bug，我们随后会解决它:

```
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1   //一旦sem满员就会自动阻塞无法进行 go func()()
        
        //我们没有必要等待协程完成就可以进行下一步代码，但是sem不是协程空间里的，所以它阻塞了，无法进行下面的协程生成
        go func() {
            process(req) // Buggy; see explanation below.
            <-sem
        }()
    }
}
```

##### 3.2.2改进方式一

The bug is that in a Go `for` loop, the loop variable is reused for each iteration, so the `req` variable is shared across all goroutines. That's not what we want. We need to make sure that `req` is unique for each goroutine. Here's one way to do that, passing the value of `req` as an argument to the closure in the goroutine:Bug出现在Go的 `for` 循环中，该循环变量在每次迭代时会被重用，因此 `req` 变量会在所有的Go程间共享，这不是我们想要的。我们需要确保 `req` 对于每个Go程来说都是唯一的。有一种方法能够做到，就是将 `req` 的值作为实参传入到该Go程的闭包中：

```
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)
    }
}
```

Compare this version with the previous to see the difference in how the closure is declared and run. 将此版本与前一版本进行比较，以查看闭包声明和运行方式的差异。Another solution is just to create a new variable with the same name, as in this example:另一个解决方案就是创建一个名字相同的新变量，如下例所示:

```
func Serve(queue chan *Request) {
    for req := range queue {
        req := req // Create new instance of req for the goroutine.
        sem <- 1
        go func() {
            process(req)
            <-sem
        }()
    }
}
```

It may seem odd to write写作可能看起来很奇怪

```
req := req
```

but it's legal and idiomatic in Go to do this. You get a fresh version of the variable with the same name, deliberately shadowing the loop variable locally but unique to each goroutine.但是这是合法的和地道的。你用相同的名字获得了该变量的一个新的版本， 以此来局部地刻意屏蔽循环变量，使它对每个Go程保持唯一。

##### 3.2.2改进方式二

Going back to the general problem of writing the server, another approach that manages resources well is to start a fixed number of `handle` goroutines all reading from the request channel.The number of goroutines limits the number of simultaneous calls to `process`. This `Serve` function also accepts a channel on which it will be told to exit; 协程数量限制了同步执行process的数量【参考协程定义】。after launching the goroutines it blocks receiving from that channel.

回到编写服务器的一般问题上来。另一种管理资源的好方法就是启动固定数量的 `handle` Go程，一起从请求信道中读取数据。Go程的数量限制了同时调用 `process` 的数量。`Serve` 同样会接收一个通知退出的信道， 在启动所有Go程后，它将阻塞并暂停从信道中接收消息。

```
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers   启动固定数量[MaxOutstanging]的Goroutines
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.  quit是一个Unbuffered的chan 需要外面告诉发送一个值，<-quit才不会阻塞。
}
```

## 4Channels of channels信道中的信道

One of the most important properties of Go is that a channel is a first-class value that can be allocated and passed around like any other. A common use of this property is to implement safe, parallel demultiplexing.

Go 最重要的属性之一是，通道是一类值，可以像任何其他值一样分配和传递。此属性的一个常见用途是实现安全的并行 解复用。

In the example in the previous section, `handle` was an idealized handler for a request but we didn't define the type it was handling. If that type includes a channel on which to reply, each client can provide its own path for the answer. Here's a `schematic` definition of type `Request`.

在上一节的示例中，句柄`handle`是一个理想化的请求处理程序，但是我们没有定义它处理的类型。如果该类型包含一个回复通道，那么每个客户端都可以为回复提供自己的路径。下面是类型 Request 的示意性定义。

```
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

The client provides a function and its arguments, as well as a channel inside the request object on which to receive the answer.

客户端提供了一个函数及其参数，以及在请求对象中接收答案的通道。

```
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
联系Serve(,)
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```

On the server side, the handler function is the only thing that changes.在服务器端，处理程序函数是唯一可以改变的东西。

```
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

There's clearly a lot more to do to make it realistic, but this code is a framework for a rate-limited, parallel, non-blocking RPC system, and there's not a mutex in sight.显然还有很多工作要做，以使其现实，但这段代码是一个速率限制、并行、非阻塞 RPC 系统的框架，而且没有看到互斥。

[RPC]: https://en.wikipedia.org/wiki/Remote_procedure_call	"go有有一个RPC包"

## Parallelization并行化

Another application of these ideas is to parallelize a calculation across multiple CPU cores. If the calculation can be broken into separate pieces that can execute independently, it can be parallelized, with a channel to signal when each piece completes.

这些想法的另一个应用是跨多个 CPU 核并行计算。如果计算可以分成独立的部分执行，那么它可以并行化，当每个部分完成时有一个通道发出信号。

Let's say we have an expensive operation to perform on a vector of items, and that the value of the operation on each item is independent, as in this idealized example.假设我们对一个项的向量执行一个开销很大的操作，并且每个项的操作值是独立的，就像这个理想化的示例一样。

```
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])  op就是Operator运算的意思
    }
    c <- 1    // signal that this piece is done
}
```

We launch the pieces independently in a loop, one per CPU. They can complete in any order but it doesn't matter; we just count the completion signals by draining the channel after launching all the goroutines.

我们在循环中启动了独立的处理块，每个CPU将执行一个处理。 它们有可能以乱序的形式完成并结束，但这没有关系； 我们只需在所有Go程开始后接收，并统计信道中的完成信号即可

```
const numCPU = 4 // number of CPU cores

func (v Vector) DoAll(u Vector) {
    c := make(chan int, numCPU)  // Buffering optional but sensible.
    for i := 0; i < numCPU; i++ {
        go v.DoSome(i*len(v)/numCPU, (i+1)*len(v)/numCPU, u, c)
    }
    // Drain the channel.
    for i := 0; i < numCPU; i++ {
        <-c    // wait for one task to complete
    }
    // All done.
}
```

Rather than create a constant value for numCPU, we can ask the runtime what value is appropriate. The function `runtime.NumCPU` returns the number of hardware CPU cores in the machine, so we could write

我们不需要为 numCPU 创建一个常量值，而是可以向`runtim询问什么值是合适的。 返回机器中硬件 CPU 内核的数量，因此我们可以编写(

```
var numCPU = runtime.NumCPU()
```

There is also a function `runtime.GOMAXPROCS`, which reports (or sets) the user-specified number of cores that a Go program can have running simultaneously. It defaults to the value of `runtime.NumCPU` but can be overridden by setting the similarly named shell environment variable or by calling the function with a positive number. Calling it with zero just queries the value. Therefore if we want to honor the user's resource request, we should write

还有一个函数 runtime.gomaxproc，它报告(或设置) Go 程序可以同时运行的用户指定的核心数量。它默认为运行时的值。但是可以通过设置类似命名的 shell 环境变量或者用正数调用函数来覆盖 NumCPU。用0调用它只会查询该值。因此，如果我们想尊重用户的资源请求，我们应该编写

```
var numCPU = runtime.GOMAXPROCS(0)
```

Be sure not to confuse the ideas of concurrency—structuring a program as independently executing components—and parallelism—executing calculations in parallel for efficiency on multiple CPUs. Although the concurrency features of Go can make some problems easy to structure as parallel computations, Go is a concurrent language, not a parallel one, and not all parallelization problems fit Go's model. For a discussion of the distinction, see the talk cited in [this blog post](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html).

确保不要混淆并发性的概念ーー将程序构造为独立执行的组件ーー和并行性ーー在多个 cpu 上并行执行计算以提高效率。虽然 Go 的并发特性可以使一些问题易于构造成并行计算，但是 Go 是一种并发语言，而不是并行语言，并不是所有的并行化问题都符合 Go 的模型。关于这两者区别的讨论，请参阅本博客文章中引用的对话。

## A leaky buffer

The tools of concurrent programming can even make non-concurrent ideas easier to express. Here's an example abstracted from an RPC package. The client goroutine loops receiving data from some source, perhaps a network. To avoid allocating and freeing buffers, it keeps a free list, and uses a buffered channel to represent it. If the channel is empty, a new buffer gets allocated. Once the message buffer is ready, it's sent to the server on `serverChan`.并发编程工具甚至可以使非并发思想更容易表达。下面是从 RPC 包中抽象出来的一个示例。客户机 goroutine 循环从某个源(可能是网络)接收数据。为了避免分配和释放缓冲区，它保留一个空闲列表，并使用一个缓冲通道来表示它。如果通道为空，则分配一个新的缓冲区。一旦消息缓冲区准备就绪，它就会发送到 serverChan 上的服务器。

```
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}
```

The server loop receives each message from the client, processes it, and returns the buffer to the free list.

服务器循环接收来自客户端的每条消息，对其进行处理，并将缓冲区返回给空闲列表。

```
func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```

The client attempts to retrieve a buffer from `freeList`; if none is available, it allocates a fresh one. The server's send to `freeList` puts `b` back on the free list unless the list is full, in which case the buffer is dropped on the floor to be reclaimed by the garbage collector. (The `default` clauses in the `select` statements execute when no other case is ready, meaning that the `selects` never block.) This implementation builds a leaky bucket free list in just a few lines, relying on the buffered channel and the garbage collector for bookkeeping.

客户端尝试从 freeList 中检索一个缓冲区; 如果没有可用的缓冲区，则分配一个新的缓冲区。除非列表已满，否则服务器发送到 freeList 将 b 放回空闲列表中，在这种情况下，缓冲区将被丢弃在地板上，由垃圾收集器回收。(select 语句中的 default 子句在没有其他情况准备好时执行，这意味着选择永远不会阻塞。)这个实现仅用几行就构建了一个无漏桶列表，依靠缓冲通道和垃圾收集器来记账。