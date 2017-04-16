

A ***first-level method*** is a method which ***DO NOT CALL *other methods* in the class/struct/interface***

A ***second-level method*** is a method which ***DO CALL *other methods* in the class/struct/interface***



Example, using structs-inheritance by embedding:

```go
    package main
    
    import "fmt"
    
    type parent struct {
    }
    func (_ parent) simple() {
    	fmt.Println("parent simple called")
    }
    func (p parent) complex() {
      fmt.Print("in complex, calling simple:")
    	p.simple()
    }
    
    type child struct {
    	parent
    }
    func (_ child) simple() {
    	fmt.Println("child simple called")
    }
    
    func main() {
    	var p = parent{}
    	var c = child{}
    
    	p.simple()
    	p.complex()
    
    	c.simple()
    	c.complex()
    }

    =>
    1 parent simple called
    2 in complex, calling simple: parent simple called
    
    3 child simple called
    4 in complex, calling simple: parent simple called //2nd level method was called on base-class context
```
     

Here `func Simple` is a first-level method (do not call other struct methods)
and `func Complex` is a second-level method (do call other struct methods)

Since all the inheritance hierarchy is made by embedding structs, the problem with 2nd level methods is that they're called in the base/parent-class context, so output line 4, where you might expect see  "child simple called" is really "parent simple called". This is correct behavior for structs, because all struct-method calls are non-virtual and resolved at compile time.


Now the example, but using interfaces:

```go
    package main
    
    import "fmt"
    
    type sampler interface {
    	simple()
    }
    
    func complex(i sampler) {
    	fmt.Print("in complex, calling simple:")
    	i.simple()
    }
    
    type parent struct {
    }
    
    func (_ parent) simple() {
    	fmt.Println("parent simple called")
    }
    
    type child struct {
    	parent
    }
    
    func (_ child) simple() {
    	fmt.Println("child simple called")
    }
    
    func main() {
    	var p = parent{}
    	var c = child{}
    
    	p.simple()
    	complex(p)
    
    	c.simple()
    	complex(c)
    }

    =>
    1 parent simple called
    2 in complex, calling simple:parent simple called
    3 child simple called
    4 in complex, calling simple:child simple called
```
    
Here `func Simple` is a first-level method (do not call other struct methods)
and `func Complex` is a second-level method (do call other *interface* methods)

Since the inheritance hierarchy of `func simple` and `func complex`is made with an *interface*, the advantage with 2nd level methods
is that they're called *by method dispatch*, so output line 4 has "child simple called".  This is correct behavior for *interfaces* because all interface-method calls are virtual so they're resolved *at run time*. 

In the first call to 2nd-level `func complex`, internal call `simple` resolved to Parent_simple. In the second call to `func complex`, internal call `simple` resolved to Child_simple. That's because all interface-method calls are virtual so they're resolved *at run time*




<small><p align=right>Written with [StackEdit](https://stackedit.io/).</p></small>
