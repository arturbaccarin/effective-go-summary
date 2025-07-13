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

## Why does Go not support overloading of methods and operators?

Experience with other languages told us that having a variety of methods with the same name but different signatures was occasionally useful but that it could also be confusing and fragile in practice. Matching only by name and requiring consistency in the types was a major simplifying decision in Go’s type system.

## Why doesn’t Go have “implements” declarations?¶

A Go type implements an interface by implementing the methods of that interface, nothing more. **_This property allows interfaces to be defined and used without needing to modify existing code_**.

## How can I guarantee my type satisfies an interface?

You can ask the compiler to check that the type T implements the interface I by attempting an assignment using the zero value for T or pointer to T, as appropriate:

```go
type T struct{}
var _ I = T{}       // Verify that T implements I.
var _ I = (*T)(nil) // Verify that *T implements I.
```

**_If T (or *T, accordingly) doesn’t implement I, the mistake will be caught at compile time_**.

## Why doesn’t type T satisfy the Equal interface?

Consider this simple interface to represent an object that can compare itself with another value:

```go
type Equaler interface {
    Equal(Equaler) bool
}
```

and this type, T:

```go
type T int
func (t T) Equal(u T) bool { return t == u } // does not satisfy Equaler
```

Unlike the analogous situation in some polymorphic type systems, T does not implement Equaler. The argument type of T.Equal is T, not literally the required type Equaler.

In Go, the type system does not promote the argument of Equal; that is the programmer’s responsibility, as illustrated by the type T2, which does implement Equaler:

```go
type T2 int
func (t T2) Equal(u Equaler) bool { return t == u.(T2) }  // satisfies Equaler
```

Even this isn’t like other type systems, though, because in Go any type that satisfies Equaler could be passed as the argument to T2.Equal, and at run time we must check that the argument is of type T2.

A related example goes the other way:

```go
type Opener interface {
   Open() Reader
}
```

func (t T3) Open() *os.File
In Go, T3 does not satisfy Opener, although it might in another language.

## Can I convert a []T to an []interface{}?

Not directly. It is disallowed by the language specification because the two types do not have the same representation in memory. It is necessary to copy the elements individually to the destination slice. This example converts a slice of int to a slice of interface{}:

```go
t := []int{1, 2, 3, 4}
s := make([]interface{}, len(t))
for i, v := range t {
    s[i] = v
}
```

## Can I convert []T1 to []T2 if T1 and T2 have the same underlying type?

```go
type T1 int
type T2 int
var t1 T1
var x = T2(t1) // OK
var st1 []T1
var sx = ([]T2)(st1) // NOT OK
```

The general rule is that you can change the name of the type being converted (and thus possibly change its method set) but you can’t change the name (and method set) of elements of a composite type. Go requires you to be explicit about type conversions.

## Why is my nil error value not equal to nil?

Under the covers, interfaces are implemented as two elements, a type T and a value V. V is a concrete value such as an int, struct or pointer, never an interface itself, and has type T. For instance, if we store the int value 3 in an interface, the resulting interface value has, schematically, (T=int, V=3). The value V is also known as the interface’s dynamic value, since a given interface variable might hold different values V (and corresponding types T) during the execution of the program.

**_An interface value is nil only if the V and T are both unset_**.

In particular, a nil interface will always hold a nil type. If we store a nil pointer of type *int inside an interface value, the inner type will be \*int regardless of the value of the pointer: (T=\*int, V=nil). **_Such an interface value will therefore be non-nil even when the pointer value V inside is nil_**.

This situation can be confusing, and arises when a nil value is stored inside an interface value such as an error return:

```go
func returnsError() error {
    var p *MyError = nil
    if bad() {
        p = ErrBad
    }
    return p // Will always return a non-nil error.
}
```

If all goes well, the function returns a nil p, so the return value is an error interface value holding (T=*MyError, V=nil). This means that if the caller compares the returned error to nil, it will always look as if there was an error even if nothing bad happened. To return a proper nil error to the caller, the function must return an explicit nil:

```go
func returnsError() error {
    if bad() {
        return ErrBad
    }
    return nil
}
```

It’s a good idea for functions that return errors always to use the error type in their signature (as we did above) rather than a concrete type such as *MyError, to help guarantee the error is created correctly.

## Why do zero-size types behave oddly?

Go supports zero-size types, such as a struct with no fields (struct{}) or an array with no elements ([0]byte). There is nothing you can store in a zero-size type, but these types are sometimes useful when no value is needed, as in map[int]struct{} or a type that has methods but no value.

**_Different variables with a zero-size type may be placed at the same location in memory. This is safe as no value can be stored in those variables._**

Moreover, **_the language does not make any guarantees as to whether pointers to two different zero-size variables will compare equal or not. Such comparisons may even return true at one point in the program and then return false at a different point_**, depending on exactly how the program is compiled and executed.

A separate issue with zero-size types is that a pointer to a zero-size struct field must not overlap with a pointer to a different object in memory. That could cause confusion in the garbage collector. This means that if the last field in a struct is zero-size, the struct will be padded to ensure that a pointer to the last field does not overlap with memory that immediately follows the struct.

## Why does Go not have variant types?

Variant types, also known as algebraic types, provide a way to specify that a value might take one of a set of other types, but only those types. A common example in systems programming would specify that an error is, say, a network error, a security error or an application error and allow the caller to discriminate the source of the problem by examining the type of the error. Another example is a syntax tree in which each node can be a different type: declaration, statement, assignment and so on.

We considered adding variant types to Go, but after discussion decided to leave them out because they overlap in confusing ways with interfaces. What would happen if the elements of a variant type were themselves interfaces?

Also, some of what variant types address is already covered by the language. The error example is easy to express using an interface value to hold the error and a type switch to discriminate cases. The syntax tree example is also doable, although not as elegantly.

## Why don’t maps allow slices as keys?

Map lookup requires an equality operator, which slices do not implement. They don’t implement equality because equality is not well defined on such types; there are multiple considerations involving shallow vs. deep comparison, pointer vs. value comparison, how to deal with recursive types, and so on. We may revisit this issue—and implementing equality for slices will not invalidate any existing programs—but without a clear idea of what equality of slices should mean, it was simpler to leave it out for now.

Equality is defined for structs and arrays, so they can be used as map keys.

## How are libraries documented?

For access to documentation from the command line, the go tool has a doc subcommand that provides a textual interface to the documentation for declarations, files, packages and so on.

The global package discovery page pkg.go.dev/pkg/. runs a server that extracts package documentation from Go source code anywhere on the web and serves it as HTML with links to the declarations and related elements. It is the easiest way to learn about existing Go libraries.

## Is there a Go programming style guide?

There is no explicit style guide, although there is certainly a recognizable “Go style”.

Go has established conventions to guide decisions around naming, layout, and file organization.

More directly, the program gofmt is a pretty-printer whose purpose is to enforce layout rules; it replaces the usual compendium of dos and don’ts that allows interpretation. All the Go code in the repository, and the vast majority in the open source world, has been run through gofmt.

## When should I use a pointer to an interface?

Almost never. Pointers to interface values arise only in rare, tricky situations involving disguising an interface value’s type for delayed evaluation.

It is a common mistake to pass a pointer to an interface value to a function expecting an interface. The compiler will complain about this error but the situation can still be confusing, because sometimes a pointer is necessary to satisfy an interface. The insight is that although **_a pointer to a concrete type can satisfy an interface, with one exception a pointer to an interface can never satisfy an interface_**.

Consider the variable declaration,

```go
var w io.Writer
```

The printing function fmt.Fprintf takes as its first argument a value that satisfies io.Writer—something that implements the canonical Write method. Thus we can write

```go
fmt.Fprintf(w, "hello, world\n")
```

If however we pass the address of w, the program will not compile.

```go
fmt.Fprintf(&w, "hello, world\n") // Compile-time error.
```