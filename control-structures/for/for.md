# For

It unifies for and while and there is no do-while. There are three forms, only one of which has semicolons.

```
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

Short declarations make it easy to declare the index variable right in the loop.

```
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

If you're looping over an array, slice, string, or map, or reading from a channel, a range clause can manage the loop.

```
for key, value := range oldMap {
    newMap[key] = value
}
```

If you **_only need the first item_** in the range (the key or index), **_drop the second_**:

```
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

If you **_only need the second item_** in the range (the value), **_use the blank identifier_**, an underscore, to discard the first:

```
sum := 0
for _, value := range array {
    sum += value
}
```

For strings, the range does more work for you, breaking out individual Unicode code points by parsing the UTF-8. 

Go doesn't just loop over bytes. Instead, it decodes the string into runes (Unicode code points) from its UTF-8 encoded bytes.

```
s := "café"
```
* 'c', 'a', 'f' = 1 byte each

* 'é' = 2 bytes in UTF-8


If the string contains invalid UTF-8 sequences, Go doesn't panic. Instead:

* It reads the invalid byte,
* Treats it as an error,
* Produces the Unicode Replacement Character U+FFFD,
* Moves on to the next byte.

```go
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}

/* prints
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
*/
```



In Go, a **_rune_** is simply:

```go
type rune = int32
```

It’s a named type representing a single Unicode code point.

Finally, Go has no comma operator and ++ and -- are statements not expressions. Thus if you want to run multiple variables in a for you should use parallel assignment (although that precludes ++ and --).

```go
// Reverse a

for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```