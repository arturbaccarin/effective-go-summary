# FAQ

## Does Go have a runtime?

> A runtime is the part of a programming language system that runs alongside your program and manages essential tasks while the program is executing. It handles things like memory allocation, garbage collection, error handling, and interacting with the operating system.

Go has an extensive runtime library, often just called the runtime, that is part of every Go program. **_This library implements garbage collection, concurrency, stack management, and other critical features of the Go language_**.

It is important to understand, however, that **_Go’s runtime does not include a virtual machine_**, such as is provided by the Java runtime. **_Go programs are compiled ahead of time to native machine code_** (or JavaScript or WebAssembly, for some variant implementations). Thus, although the term is often used to describe the virtual environment in which a program runs, in Go the word “runtime” is just the name given to the library providing critical language services.


## Why does Go not have exceptions?

We believe that coupling exceptions to a control structure, as in the try-catch-finally idiom, results in convoluted code. It also tends to encourage programmers to label too many ordinary errors, such as failing to open a file, as exceptional.

> Convoluted code is code that is overly complex, hard to read, and difficult to understand. It often has tangled logic, too many nested structures, or unclear flow, making it challenging for developers to follow or maintain.

Go takes a different approach. For plain error handling, Go’s multi-value returns make it easy to report an error without overloading the return value.

## Why goroutines instead of threads?

Goroutines are part of making concurrency easy to use. The idea, which has been around for a while, **_is to multiplex independently executing functions—coroutines—onto a set of threads. When a coroutine blocks, such as by calling a blocking system call, the run-time automatically moves other coroutines on the same operating system thread to a different, runnable thread so they won’t be blocked_**. The programmer sees none of this, which is the point. The result, which we call goroutines, can be very cheap: they have little overhead beyond the memory for the stack, which is just a few kilobytes.

## Is Go an object-oriented language?

Yes and no. Although Go has types and methods and allows an object-oriented style of programming, there is no type hierarchy. The concept of “interface” in Go provides a different approach that we believe is easy to use and in some ways more general. There are also ways to embed types in other types to provide something analogous—but not identical—to subclassing.

## Why is there no type inheritance?

Object-oriented programming, at least in the best-known languages, involves too much discussion of the relationships between types, relationships that often could be derived automatically. Go takes a different approach.

Rather than requiring the programmer to declare ahead of time that two types are related, in Go a type automatically satisfies any interface that specifies a subset of its methods. Besides reducing the bookkeeping, this approach has real advantages. Types can satisfy many interfaces at once, without the complexities of traditional multiple inheritance. Interfaces can be very lightweight—an interface with one or even zero methods can express a useful concept. Interfaces can be added after the fact if a new idea comes along or for testing—without annotating the original types. Because there are no explicit relationships between types and interfaces, there is no type hierarchy to manage or discuss.