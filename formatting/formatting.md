# Formatting

With Go we take an unusual approach and let the machine take care of most formatting issues. The gofmt program (also available as go fmt, which operates at the package level rather than source file level) reads a Go program and emits the source in a standard style of indentation and vertical alignment, retaining and if necessary reformatting comments.

As an example, there's no need to spend time lining up the comments on the fields of a structure. Gofmt will do that for you. Given the declaration:

```go
type T struct {
    name string // name of the object
    value int // its value
}

type T struct {
    name    string // name of the object
    value   int    // its value
}
```

Some formatting details remain. Very briefly:

* Indentation: We use **tabs** for indentation and gofmt emits them by default. Use spaces only if you must.

* Line length: Go has **no line length limit**. Don't worry about overflowing a punched card. If a line feels too long, wrap it and indent with an extra tab.

* Parentheses: Go needs fewer parentheses than C and Java: control structures (if, for, switch) do not have parentheses in their syntax.