### Type Switch

A golang *Type Switch* has a lot of magic in it. 
By using the create and assign syntax (*:=*) inside a *Type Switch*
you're defining and assigning the same var with different types on each case.

Example from [go-wiki](https://code.google.com/p/go-wiki/wiki/Switch)

```go
    func do(v interface{}) string {
            switch u := v.(type) {
            case int:
                    return strconv.Itoa(u*2) // u has type int
            
            case string:
                    mid := len(u) / 2 // split - u has type string
                    return u[mid:] + u[:mid] // join
            
            case Stringer: //*another (non-empty) interface*
                    return u.String() //call via method dispatch
            }
            return "unknown"
    }
  
    =>
    do(21) == "42"
    do("bitrab") == "rabbit"
    do(3.142) == "unknown"
  
  Pseudo-code
  
    func do(v interface{}) string 
  
        switch type-in.(v)
            
            case int:
                    var u int = value-in.(v)
                    return strconv.Itoa(u*2) 
            
            case string:
                    var u string = value-in.(v)
                    let mid = len(u) / 2
                    return u[mid:] + u[:mid] 
  
            case Stringer-interface: 
                    var u Stringer-interface = value-in.(v)
                    return u.String() //call via method dispatch
            
       return "unknown"
 ```
  
