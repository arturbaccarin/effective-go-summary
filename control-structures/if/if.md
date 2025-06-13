# IF

In Go a simple if looks like this:

```
if x > 0 {
    return y
}
```

Mandatory braces encourage writing simple if statements on multiple lines. It's good style to do so anyway, especially when the body contains a control statement such as a return or break.

Since if and switch accept an initialization statement, it's common to see one used to set up a local variable.

```
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

In the Go libraries, you'll find that when an if statement doesn't flow into the next statement—that is, the body ends in break, continue, goto, or return—the unnecessary else is omitted.

```
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

This is an example of a common situation where code must guard against a sequence of error conditions. The code reads well if the successful flow of control runs down the page, eliminating error cases as they arise. Since error cases tend to end in return statements, the resulting code needs no else statements.

```
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

## Redeclaration and reassignment

The declaration that calls os.Open reads,

```
f, err := os.Open(name)
```

This statement declares two variables, f and err. A few lines later, the call to f.Stat reads,

```
d, err := f.Stat()
```

which looks as if it declares d and err. Notice, though, that err appears in both statements. This duplication is legal: err is declared by the first statement, **_but only re-assigned in the second_**. This means that the call to f.Stat uses the existing err variable declared above, and just gives it a new value.