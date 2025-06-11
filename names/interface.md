# Interface names¶

By convention, one-method interfaces are **_named by the method name plus an -er suffix_** or similar modification to construct an agent noun: `Reader, Writer, Formatter, CloseNotifier etc.`

There are a number of such names and it's productive to honor them and the function names they capture. Read, Write, Close, Flush, String and so on have canonical signatures and meanings. 

| Function | Expected Meaning                   | Interface         |
| -------- | ---------------------------------- | ----------------- |
| `Read`   | Reads data into a buffer           | `io.Reader`       |
| `Write`  | Writes data from a buffer          | `io.Writer`       |
| `Close`  | Releases resources                 | `io.Closer`       |
| `Flush`  | Forces buffered data to be written | `flusher` pattern |
| `String` | Returns a string representation    | `fmt.Stringer`  

To avoid confusion, don't give your method one of those names unless it has the same signature and meaning. 

Conversely, if your type implements a method with the same meaning as a method on a well-known type, give it the same name and signature; **_call your string-converter method String not ToString_**.

