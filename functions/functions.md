# Functions

## Multiple return values

One of Go's unusual features is that functions and methods can return multiple values. This form can be used to improve on a couple of clumsy idioms in C programs: in-band error returns such as -1 for EOF and modifying an argument passed by address.

In C, a Write error is signaled by a negative count with the error code secreted away in a volatile location.

In Go, Write can return a count and an error: “Yes, you wrote some bytes but not all of them because you filled the device”. The signature of the Write method on files from package os is:

```go
func (file *File) Write(b []byte) (n int, err error)
```

and as the documentation says, it returns the number of bytes written and a non-nil error when n != len(b).

A similar approach obviates the need to pass a pointer to a return value to simulate a reference parameter. Here's a simple-minded function to grab a number from a position in a byte slice, returning the number and the next position.

```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

## Named result parameters
The return or result "parameters" of a Go function can be given names and used as regular variables, just like the incoming parameters. When named, they are initialized to the zero values for their types when the function begins; if the function executes a return statement with no arguments, the current values of the result parameters are used as the returned values.

The names are not mandatory but they can make code shorter and clearer: they're documentation. If we name the results of nextInt it becomes obvious which returned int is which.

```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

## Defer
Go's defer statement schedules a function call (the deferred function) to be run immediately before the function executing the defer returns.

It's an unusual but **_effective way to deal with situations such as resources that must be released regardless of which path a function takes to return_**. The canonical examples are unlocking a mutex or closing a file.

```go
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
```

Deferring a call to a function such as Close has two advantages:
* First, it guarantees that you will never forget to close the file, a mistake that's easy to make if you later edit the function to add a new return path.
* Second, it means that the close sits near the open, which is much clearer than placing it at the end of the function.

The arguments to the deferred function (which include the receiver if the function is a method) **_are evaluated when the defer executes, not when the call executes_**.

Besides avoiding worries about variables changing values as the function executes, this means that a single deferred call site can defer multiple function executions.

Deferred functions are executed in LIFO order, so this code will cause 4 3 2 1 0 to be printed when the function returns.
```go
func main() {
	for i := 0; i < 5; i++ {
		defer fmt.Printf("%d ", i)
	}

	fmt.Println("Hello, 世界")
}

// out: Hello, 世界
// 4 3 2 1 0 
```

A more plausible example is a simple way to trace function execution through the program. We could write a couple of simple tracing routines like this:

```go
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

We can do better by exploiting the fact that **_arguments to deferred functions are evaluated when the defer executes_**. The tracing routine can set up the argument to the untracing routine. This example:

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}

/*
entering: b
in b
entering: a
in a
leaving: a
leaving: b
*/
```