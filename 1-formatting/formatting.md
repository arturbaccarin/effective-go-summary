# Formatting

With Go we take an unusual approach and let the machine take care of most formatting issues. The gofmt program (also available as go fmt, which operates at the package level rather than source file level) reads a Go program and emits the source in a standard style of indentation and vertical alignment, retaining and if necessary reformatting comments.

As an example, there's no need to spend time lining up the comments on the fields of a structure. Gofmt will do that for you. Given the declaration:

```
type T struct {
    name string // name of the object
    value int // its value
}

type T struct {
    name    string // name of the object
    value   int    // its value
}
```