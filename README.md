## Documents: (see files in this repository)

[Golang Struct and Interface from OOP concepts](OOP.md)

## Loose questions

**Why "flag" package returns pointers or require references?**

usage:

```go
    import "flag"
    var int_ptr = flag.Int("flagname", 1234, "help message for flagname")

    package flag  
    func Int(name string, value int, usage string) *int
```
   
- because they make all the assignments at the flags.Parse() function.

## Gotchas

**Interface**

An Interface is a type holding two pointers: 
* a pointer to a method table (holding type and method implementations)
* a pointer to a concrete value (the type defined by the method table)

Ref:
- http://research.swtch.com/interfaces
- http://blog.golang.org/laws-of-reflection

**Compare Interface to nil**

Comparing a `var:Interface` to `nil` evaluates to *true* **only if both pointers are 0 (null pointers)**

**Why is my nil error value not equal to nil?**
http://golang.org/doc/faq#nil_error

Under the covers, interfaces are implemented as two elements, a type and a value. The value, called the interface's dynamic value, is an arbitrary concrete value and the type is that of the value. For the int value 3, an interface value contains, schematically, (int, 3).

An interface value is nil only if the inner value and type are both unset, (nil, nil). In particular, a nil interface will always hold a nil type. If we store a pointer of type *int inside an interface value, the inner type will be *int regardless of the value of the pointer: (*int, nil). Such an interface value will therefore be non-nil even when the pointer inside is nil.

This situation can be confusing, and often arises when a nil value is stored inside an interface value such as an error return:

```go
    func returnsError() error {
    	var p *MyError = nil
    	if bad() {
    		p = ErrBad
    	}
    	return p // Will always return a non-nil error.
    }
```
    
If all goes well, the function returns a nil p, so the return value is an error interface value holding (*MyError, nil). This means that if the caller compares the returned error to nil, it will always look as if there was an error even if nothing bad happened. To return a proper nil error to the caller, the function must return an explicit nil:

```go

    func returnsError() error {
    	if bad() {
    		return ErrBad
    	}
    	return nil
    }
```
    
It's a good idea for functions that return errors always to use the error type in their signature (as we did above) rather than a concrete type such as *MyError, to help guarantee the error is created correctly. As an example, os.Open returns an error even though, if not nil, it's always of concrete type *os.PathError.

Similar situations to those described here can arise whenever interfaces are used. Just keep in mind that if any concrete value has been stored in the interface, the interface will not be nil. For more information, see The Laws of Reflection.
