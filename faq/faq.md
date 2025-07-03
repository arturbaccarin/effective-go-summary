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