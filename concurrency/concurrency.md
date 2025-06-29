# Concurrency

Go encourages a different approach in which **_shared values are passed around on channels_** and, in fact, never actively shared by separate threads of execution. **_Only one goroutine has access to the value at any given time. Data races cannot occur, by design. To encourage this way of thinking we have reduced it to a slogan_**:

> Do not communicate by sharing memory; instead, share memory by communicating.

* Only one goroutine "owns" the data at a time, so others can't mess with it.
* This design prevents data races, which happen when two goroutines try to change the same data at the same time.

This approach can be taken too far. Reference counts may be best done by putting a mutex around an integer variable, for instance. But as a high-level approach, using channels to control access makes it easier to write clear, correct programs.

In Go, **_the recommended way for different parts of a program (called goroutines) to work together is by sending data through channels, instead of having them share access to the same data_**.

## Goroutines

**_They're called goroutines because the existing terms—threads, coroutines, processes, and so on—convey inaccurate connotations_**. 

A goroutine has a simple model: **_it is a function executing concurrently with other goroutines in the same address space_**. It is lightweight, costing little more than the allocation of stack space. And the stacks start small, so they are cheap, and grow by allocating (and freeing) heap storage as required.

**_Goroutines are multiplexed onto multiple OS threads so if one should block, such as while waiting for I/O, others continue to run_**. Their design hides many of the complexities of thread creation and management.

Prefix a function or method call with the go keyword to run the call in a new goroutine. When the call completes, the goroutine exits, silently. (The effect is similar to the Unix shell's & notation for running a command in the background.)

```go
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

A function literal can be handy in a goroutine invocation.

```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

## Channels

Like maps, channels are allocated with make, and the resulting value acts as a reference to an underlying data structure.

```go
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

**_Unbuffered channels_** combine communication—the exchange of a value—with **_synchronization—guaranteeing_** that two calculations (goroutines) are in a known state.

A channel can allow the launching goroutine to wait for the sort to complete.

```go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

**_Receivers always block until there is data to receive. If the channel is unbuffered, the sender blocks until the receiver has received the value_**. If the channel has a buffer, the sender blocks only until the value has been copied to the buffer; if the buffer is full, this means waiting until some receiver has retrieved a value.

**_A buffered channel can be used like a semaphore, for instance to limit throughput_**. In this example, **_incoming requests are passed to handle, which sends a value into the channel, processes the request, and then receives a value from the channel to ready the “semaphore” for the next consumer_**. The capacity of the channel buffer limits the number of simultaneous calls to process.

Once MaxOutstanding handlers are executing process, any more will block trying to send into the filled channel buffer, until one of the existing handlers finishes and receives from the buffer.

```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    sem <- 1    // Wait for active queue to drain.
    process(r)  // May take a long time.
    <-sem       // Done; enable next request to run.
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // Don't wait for handle to finish.
    }
}
```

This design has a problem, though: Serve creates a new goroutine for every incoming request, even though only MaxOutstanding of them can run at any moment. As a result, the program can consume unlimited resources if the requests come in too fast. 

We can address that deficiency by changing Serve to gate the creation of the goroutines:

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1 // Wait for space in the semaphore (limit active handlers).
        go func() {
            process(req)
            <-sem // Release the slot in the semaphore after processing.
        }()
    }
}
```

Another approach that manages resources well is to start a fixed number of handle goroutines all reading from the request channel. The number of goroutines limits the number of simultaneous calls to process. This Serve function also accepts a channel on which it will be told to exit; after launching the goroutines it blocks receiving from that channel.

```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```

## Channels of channels

One of the most important properties of Go is that a channel is a first-class value that can be allocated and passed around like any other. A common use of this property is to implement safe, parallel demultiplexing.

In the example in the previous section, handle was an idealized handler for a request but we didn't define the type it was handling. If that type includes a channel on which to reply, each client can provide its own path for the answer. Here's a schematic definition of type Request.

```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

The client provides a function and its arguments, as well as a channel inside the request object on which to receive the answer.

```go
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```

On the server side, the handler function is the only thing that changes.

```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

## Parallelization

Another application of these ideas is to parallelize a calculation across multiple CPU cores. 

If the calculation can be broken into separate pieces that can execute independently, it can be parallelized, with a channel to signal when each piece completes.

Let's say we have an expensive operation to perform on a vector of items, and that the value of the operation on each item is independent, as in this idealized example.

```go
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // signal that this piece is done
}
```