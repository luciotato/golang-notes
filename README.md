##Documents: (see files in this repository)

[Golang Struct and Interface from OOP concepts](OOP.md)

##Loose questions

**Why "flag" package returns pointers or require references?**

usage:

    import "flag"
    var int_ptr = flag.Int("flagname", 1234, "help message for flagname")

    package flag  
    func Int(name string, value int, usage string) *int
   
- because they make all the assignments at the flags.Parse() function.

