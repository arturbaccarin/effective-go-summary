# Interfaces and other types

## Interfaces

Interfaces in Go provide a way to specify the behavior of an object: **_if something can do this, then it can be used here_**.

Interfaces with only one or two methods are common in Go code, and are usually given a name derived from the method, such as io.Writer for something that implements Write.

**_A type can implement multiple interfaces_**.

For instance, a collection can be sorted by the routines in package sort if it implements sort.Interface, which contains Len(), Less(i, j int) bool, and Swap(i, j int), and it could also have a custom formatter. In this contrived example Sequence satisfies both.

```go
type Sequence []int

// Methods required by sort.Interface.
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Copy returns a copy of the Sequence.
func (s Sequence) Copy() Sequence {
    copy := make(Sequence, 0, len(s))
    return append(copy, s...)
}

// Method for printing - sorts the elements before printing.
func (s Sequence) String() string {
    s = s.Copy() // Make a copy; don't overwrite argument.
    sort.Sort(s)
    str := "["
    for i, elem := range s { // Loop is O(N²); will fix that in next example.
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```

## Conversions

The String method of Sequence is recreating the work that Sprint already does for slices. (It also has complexity O(N²), which is poor.) We can share the effort (and also speed it up) if we convert the Sequence to a plain []int before calling Sprint.

```go
func (s Sequence) String() string {
    s = s.Copy()
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```

This method is another example of the conversion technique for calling Sprintf safely from a String method. Because the two types (Sequence and []int) are the same if we ignore the type name, it's legal to convert between them. **_The conversion doesn't create a new value, it just temporarily acts as though the existing value has a new type_**. (There are other legal conversions, such as from integer to floating point, that do create a new value.)

It's an idiom in Go programs to convert the type of an expression to access a different set of methods. As an example, we could use the existing type sort.IntSlice to reduce the entire example to this:

```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    s = s.Copy()
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

Now, instead of having Sequence implement multiple interfaces (sorting and printing), we're using the ability of a data item to be converted to multiple types (Sequence, sort.IntSlice and []int), each of which does some part of the job. That's more unusual in practice but can be effective.