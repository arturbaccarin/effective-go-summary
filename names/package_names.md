# Package Names
When a package is imported, the package name becomes an accessor for the contents.

In `import "bytes"` the importing package can talk about _bytes.Buffer_.

The package name should be good: **_short, concise, evocative_**.

By convention, packages are given **_lower case, single-word names_**; there should be **_no need for underscores or mixedCaps_**.

And don't worry about collisions a priori. The package name is only the default name for imports; **_it need not be unique across all source code_**, and in the rare case of a collision the importing package can choose a different name to use locally.

Another convention is that **_the package name is the base name of its source directory_**; the package in **_src/encoding/base64 is imported as "encoding/base64" but has name base64, not encoding_base64 and not encodingBase64_**.

Don't use the `import .` notation:
> The `import .` notation is a special import form that imports all the exported names from a package directly into the current namespace, so you can use them without a prefix.

```go
import . "fmt"

func main() {
    Println("Hello, world") // You can call Println directly, no fmt.Printin
}
```

While it might seem convenient, `import .` is discouraged for several reasons:

* Readability: It becomes unclear where a function or type comes from.

* Name conflicts: If two packages export the same name, you can get collisions.

* Maintenance issues: It’s harder to figure out dependencies or to refactor code.

It can simplify tests that must run outside the package they are testing, but should otherwise be avoided.

```go
package mypkg_test

import (
    . "myorg/mypkg"  // Allows calling Foo() instead of mypkg.Foo()
    "testing"
)

func TestSomething(t *testing.T) {
    result := Foo() // Foo is from mypkg
    if result != "bar" {
        t.Fail()
    }
}
```

The importer of a package will use the name to refer to its contents, so exported names in the package can use that fact to avoid repetition. 

For instance, the buffered reader type in the **_bufio package_** is called `Reader`, not `BufReader`, because users see it as `bufio.Reader`, which is a clear, concise name.

Moreover, because imported entities are always **_addressed with their package name, bufio.Reader does not conflict with io.Reader_**.

Similarly, the function to make new instances of `ring.Ring` — which is the definition of a constructor in Go — **_would normally be called NewRing_**, but since Ring is the only type exported by the package, and since the package is called ring, **_it's called just New, which clients of the package see as ring.New_**. Use the package structure to help you choose good names.
