# Semicolons

Like C, **_Go's formal grammar uses semicolons to terminate statements**, but unlike in C, **those semicolons do not appear in the source_**. 

Instead **_the lexer (compiler) uses a simple rule to insert semicolons automatically as it scans, so the input text is mostly free of them_**.

The rule is this:
> If the last token before a newline is:
> * an identifier (e.g., a variable name, int, float64, etc.),
> * a basic literal (e.g., a number like 123, or a string like "hello"),
> * or a token like ), ], or }
>
> Then the Go lexer  will automatically insert a semicolon there.



This could be summarized as:
> if the newline comes after a token that could end a statement, insert a semicolon.

A semicolon can also be omitted immediately before a closing brace, so a statement such as:

`go func() { for { dst <- <-src } }()`

 Idiomatic Go programs **_have semicolons_** only in places such as for loop clauses, to separate the initializer, condition, and continuation elements. They are also necessary to separate multiple statements on a line, should you write code that way.

 One consequence of the semicolon insertion rules is that **_you cannot put the opening brace of a control structure (if, for, switch, or select) on the next line_**. If you do, a semicolon will be inserted before the brace, which could cause unwanted effects. Write them like this:

 ```go
 if i < f() {
    g()
}
 ```

 not like this

 ```go
 if i < f()  // wrong!
{           // wrong!
    g()
}
 ```