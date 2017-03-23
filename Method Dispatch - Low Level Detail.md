***This is what I understand happens at low-level when you use interface parameters/vars***

### Golang Interfaces

A ***golang-Interface*** is ***a class with no fields and ONLY VIRTUAL methods***.

```go
    type J interface 
       to-string() ( string )
       dosomea ( x, J )
       dosomeb ( x, k ) (J) 
       dosomec ( x, a string) ( int )
```

If a struct *implements* such methods, it “satisfies the interface”.

You can create a “var of type interface” or declare a function param of type interface.
* a `var x` of type interface is NIL until a *concrete-type* its assigned to it.
* a `var x` of type interface is internally just 2 pointers.  One pointer to an ITable and other to the concrete value stored. 

so `var x J-interface` internally is:

```go
    c-struct interface-var
		ITable            *ITable
		concrete-value    any_union
```

an ITable is:

```go
    c-struct ITable
           concrete-type  *type
       	   interface-type *interface
           jmp-table[]    *function
```

      
When a concrete type is assigned to a interface-var, golang *creates a jmp-method table on the fly (and caches it). There is a jmp-table for each combination of J-Interface \* concrete-type-implementing-J-interface

Let’s assume z is struct Z and there is a bunch of functions as *func dosomex ( z ){...}* so that struct Z implements interface J

When you do:

```go
     var x J-interface  = z
 ```

golang *constructs the ITable on the fly*. The J-interface has 4 methods, so it needs to calculate x.ITable. Golang will search the method “to-string” of struct Z and put it in ITable.jmp-table[0], then look for “dosomea (struct Z)” and put it in ITable.jmp-table[1] and so on. Then it will store ITable->concrete = &struct-Z, and X.itable->interface= &interface-J”.  

x, being an *interface* type, has two 32bit pointers, the first will point to the newly created ITable “&struct-Z, &interface-J, jmp-table[0...3]” (where the concrete-type-info is stored) and the second points to the concrete-value (in this case a copy of z)

then…

```go
    for n:=1; n++<100 {
         print( x.dosomea(x))
    }
```

the call *x.dosomea()* is: 
* x is var type interface -> struct Z, Interface J
* so x.ITable -> “struct-Z,interface-J,jmp-table[0..3]”
* *x.dosomea()* gets translated to call [x.ITable.jmps[1]] (since “dosomea” => index 1)
* this is a memory fetch and an indirect-call.
* interface values are passed by value, so x.ITable and x.value are always available in the stack. To know the concrete type stored in x.value golang must dereference c.ITable->concrete-type.

