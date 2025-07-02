# Errors

As mentioned earlier, Go's multivalue return makes it easy to return a detailed error description alongside the normal return value. 

**_It is good style to use this feature to provide detailed error information_**. For example, as we'll see, os.Open doesn't just return a nil pointer on failure, it also returns an error value that describes what went wrong.

By convention, errors have type error, a simple built-in interface.

```go
type error interface {
    Error() string
}
```

A library writer is free to implement this interface with a richer model under the covers, making it possible not only to see the error but also to provide some context.

As mentioned, alongside the usual *os.File return value, os.Open also returns an error value. If the file is opened successfully, the error will be nil, but when there is a problem, it will hold an os.PathError:

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

Such an error, which includes the problematic file name, the operation, and the operating system error it triggered, is useful even if printed far from the call that caused it; it is much more informative than the plain "no such file or directory".

When feasible, **_error strings should identify their origin, such as by having a prefix naming the operation or package that generated the error_**. For example, **_in package image, the string representation for a decoding error due to an unknown format is "image: unknown format"_**.

Callers that care about the precise error details can use a type switch or a type assertion to look for specific errors and extract details.

```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

## Panic

The usual way to report an error to a caller is to return an error as an extra return value. The canonical Read method is a well-known instance; it returns a byte count and an error.

But what if the error is unrecoverable? Sometimes the program simply cannot continue.

For this purpose, there is a **_built-in function panic that in effect creates a run-time error that will stop the program_**.
The function **_takes a single argument of arbitrary type—often a string—to be printed as the program dies_**.

It's also a way **_to indicate that something impossible has happened_**, such as exiting an infinite loop.

```go
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

This is only an example but **_real library functions should avoid panic. If the problem can be masked or worked around, it's always better to let things continue to run rather than taking down the whole program. One possible counterexample is during initialization: if the library truly cannot set itself up, it might be reasonable to panic, so to speak_**.

## Recover

**_When panic is called_**, including implicitly for run-time errors such as indexing a slice out of bounds or failing a type assertion, it **_immediately stops execution of the current function and begins unwinding the stack of the goroutine, running any deferred functions along the way_**. If that unwinding reaches the top of the goroutine's stack, the program dies. 

However, **_it is possible to use the built-in function recover to regain control of the goroutine and resume normal execution_**.

**_A call to recover stops the unwinding and returns the argument passed to panic_**.

**_Because the only code that runs while unwinding is inside deferred functions, recover is only useful inside deferred functions_**.

```go
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

In this example, if do(work) panics, the result will be logged and the goroutine will exit cleanly without disturbing the others. There's no need to do anything else in the deferred closure; calling recover handles the condition completely.

Because recover always returns nil unless called directly from a deferred function, deferred code can call library routines that themselves use panic and recover without failing. As an example, the deferred function in safelyDo might call a logging function before calling recover, and that logging code would run unaffected by the panicking state.

With our recovery pattern in place, the do function (and anything it calls) can get out of any bad situation cleanly by calling panic. We can use that idea to simplify error handling in complex software.

```go
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
    return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```

If doParse panics, the recovery block will set the return value to nil—deferred functions can modify named return values. It will then check, in the assignment to err, that the problem was a parse error by asserting that it has the local type Error. If it does not, the type assertion will fail, causing a run-time error that continues the stack unwinding as though nothing had interrupted it.

This check means that if something unexpected happens, such as an index out of bounds, the code will fail even though we are using panic and recover to handle parse errors.