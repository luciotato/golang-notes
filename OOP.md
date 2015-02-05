#Golang concepts from an OOP point of view

##Objective

The objective of this document is to ease golang learning by associating
golang-specific *concepts* with previously known concepts in the OOP field.

This is a discovery process, I'm writing this document to help myself understanding golang and maybe help others.

***The objective of this document is to promote golang usage*** by easing language understanding for people coming from a heavy OOP background

##Introduction
Golang introduce words with a new golang-specific meaning, as *struct* and *interface*.  
This is not bad, but sometimes is nice to have a "translation" available to be able to understand golang-concepts by relating them to previously known concepts.

This is important in order to *understand* concepts of a new language, 
if you can *translate* a golang-word to previously known concepts, 
the learning is by far easier.

##Why

This is a discovery process. I'm just starting to analyze golang. 

This is a work in progress, this document is published in github because ***I'm expecting pull requests from people with a better understanding of golang internals.***

##Cheat Sheet

|golang concept|OOP concept|
|-----|-----|
|*struct*|*class*
|*embedding*|*multiple inheritance*
|*receiver*|*this*
|*interface*|*class with no fields and only abstract methods*

##golang-struct

A ***golang-struct*** is a ***class*** with ***fields***. e.g.:

  type Rectangle struct {
    Name          string
    Width, Height float64
  }
  
This can be read as (pseudo code):

    class Rectangle
      field name: string
      field Width: float64
      field Height: float64

***Notes:***
  
- There is a *zero-value* defined for each core-type, so if you do not provide values at instantiation, all fields will have the zero-value

- No specific in-class constructors. There is a *generic constructor* for any class instance, receiving a generic literal in a JSON-like format and using reflection to create instances of any class.


        // generic constructor: pseudo-code, recursive
        function construct ( class,  literal) 

          helper function assign ( object, literal)  //recursive
            if type-of object is "object"
                for each field in object.fields
                    if literal has-field field.name 
                        assign(field.value,  literal[field.name].value) //recurse
            else
                set object.value = literal.value
          

        // generic constructor main body
        set class-instance = new class
        assign class-instance, literal
    
    
Example:

    package main
    import . "fmt"
    
    type Rectangle struct {
      Name          string
      Width, Height float64
    }
    
    func main() {
    
      var a Rectangle
      var b = Rectangle{"I'm b.", 10, 20}
      var c = Rectangle{Height: 12, Width: 14}
    
      Println(a)
      Println(b)
      Println(c)
    }

    =>
    { 0 0}
    {I'm b. 10 20}
    { 14 12}

##golang "embedded field" is ***multiple inheritance***

***multiple inheritance*** is achieved in golang by *embedding* a field ***named as other class*** inside a struct. 

Note: The official documents call it "an annonymous field", but this add to confusion, since the field *has a name*: the inherited class-name, and can be accessed by dot notation.

    type NamedObj struct {
      Name      string
    }
  
    type Shape struct {
      color     int32
      isRegular bool
    }
    
    type Point struct {
      x,y float64
    }
  
    type Rectangle struct {
      NamedObj           //inheritance
      Shape              //inheritance
      center Point       //standard composition
      Width, Height float64
    }

This can be read: (pseudo code)

    class NamedObj
       field Name: string
  
    class Shape
       field color: int32
       field isRegular: bool
       
    class Rectangle
       inherits NamedObj
       inherits Shape
       field center: Point
       field Width: float64
       field Height: float64
  
    var aRect Rectangle
  
Since we're using class-named-fields for inheritance, we can use the field to access the base-class. So in `aRect`: 

***aRect.Name*** and ***aRect.NamedObj.Name*** refer to the same field

***aRect.color*** and ***aRect.Shape.color*** refer to the same field

###Method overriding

If you have a ***method show()*** for example in ***class NamedObj*** and also define a ***method show()*** in ***class Rectangle***,
***Rectangle_show()*** will override ***NamedObj_Show()***

As with base class fields, you can use the ***inherited class-name-as-field*** to access the base implementation via dot-notation, e.g.:

    a.show()          // calls a.Rectangle_show()
    a.NamedObj.show() // calls a.NamedObj_show(), the base implementation


###Multiple inheritance and The Diamond Problem

Golang solves [the diamond problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem) by *not allowing diamonds*.

Since ***inheritance (embedded fields)*** include inherited field names in the ***inheriting class (struct)***, all *embedded* class-field-names *should not collide*. You must rename fields if there is a name collision. This rule avoids the diamond problem, by not allowing it.

##Golang *methods* and ***"receivers" (this)***

A golang ***method*** is the same as a ***class method*** but:

- It is defined *outside* of the ***class(struct)*** body
- Since it is outside the class, it has an *extra section* before the method name to define the ***"receiver" (this)***. 
- The extra section defines ***this*** as an ***explicit parameter*** (The ***this/self*** parameter is implicit in most OOP languages).
- Since there is such a special section to define ***this (receiver)***, you can also select a ***name*** for ***this/self***. Idiomatic golang is to use a short var name with the class initials. e.g.:

Example

        //class NamedObj
        type NamedObj struct {
          Name      string
        }
        //method show
        func (n NamedObj) show() {
          Println(n.Name)  // "n" is "this"
        }
      
        //class Rectangle
        type Rectangle struct {
          NamedObj              //inheritance
          Width, Height float64
        }
        //override method show
        func (r Rectangle) show() {
          Println("Rectangle ",r.name)  // "r" is "this"
        }

  Pseudo-code:
  
      class NamedObj
         Name: string
    
         method show
           print this.Name
    
      class Rectangle
         inherits NamedObj
         field Width: float64
         field Height: float64
    
         method show //override
           print "Rectangle", this.Name

Using it:

    func main() {
  
      var a = NamedObj{"Joe"}
      var b = Rectangle{NamedObj{"Richard"}, 10, 20}
  
      a.show("Hello")
      b.show("Hello")
  
    }

    =>
    Hello I'm Joe
    Hello I'm Richard
    - I'm a Rectangle named Richard


##a golang-Interface is an *class with no fields and only abstract methods*

A ***golang-Interface*** is ***class with no fields and only abstract methods***.
Given this definition, you can use an interface to:

- *implement* an interface, by declaring all the *interface abstract methods* in a *concrete class (a struct)*
- *inherit(embed)* a golang-interface into another golang-interface

- Declare a var or parameter of type *interface*.

###Declare a var/parameter with type interface

By picturing an ***Interface*** as a ***class with no fields and only abstract methods***, you can understand the advantages and limitations of *Interfaces*

By declaring a var/parameter with type *interface* you define the set of valid methods for the var/parameter. This allows you to use a form of ***polymorphism*** via ***method dispatch***

When you declare a var/parameter with type interface:

- The var/parameter do not have fields
- The var/parameter has *a defined set of methods*
- When you call a method of the var/parameter, a *concrete method* is called via *method dispatch* from a jmp-table. 
- When used as parameter type: 
  - You can call the function with any class implementing the interface
  - The function works for every class implementing the interface (polymorphic)

Note: The *ITables* used for method dispatch are constructed dynamically as needed and cached. Each *class(struct)* has one ITable for each *Interface* the *class(struct) implements*), so, if all *classes(structs)* implement all interfaces, there's a *ITable* for each *class(struct)*-*Interface* combination. See: [Go Data Structures: Interfaces](http://research.swtch.com/interfaces)

Note: I'm writing a [low-level detail of what happens when you use interface vars](low-level-interface.md). Help is appreciated.

####Examples from [How to use interfaces in Go](http://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go) with commented pseudo-code

    package main
    
    import (
      "fmt"
    )
    
    /*
    class Animal
       abstract Speak() string
    */
    type Animal interface {
      Speak() string
    }
    
    /*
    class Dog
      method Speak() string
         return "Woof!"
    */       
    type Dog struct {
    }
    func (d Dog) Speak() string {
      return "Woof!"
    }
    
    /*
    class Cat
      method Speak() string
         return "Meow!"
    */       
    type Cat struct {
    }
    func (c Cat) Speak() string {
      return "Meow!"
    }
    
    /*
    class Llama
      method Speak() string
         return "LaLLamamQueLLama!"
    */       
    type Llama struct {
    }
    func (l Llama) Speak() string {
      return "LaLLamamQueLLama!"
    }
    
    /*
    func main
      var animals = [ Dog{}, Cat{}, Llama{} ]
      for each animal in animals
         print animal.Speak() // method dispatch via jmp-table
    */       
  
    func main() {
      animals := []Animal{Dog{}, Cat{}, Llama{}}
      for _, animal := range animals {
        fmt.Println(animal.Speak()) // method dispatch via jmp-table
      }
    }

###The empty Interface

By picturing an ***Interface*** as a ***class with no fields and only abstract methods***, you can understand ***what is the golang empty interface: "Interface{}"***.

***Interface{}*** in golang is a ***class with no fields and no methods***

What can you do with a `var Interface{}`? Well, initialy, nothing.

But, since by definition all *classes(structs)* implement *Interface{}* it means
that a `var Interface{}` ***can hold any value***

To actually use the *value* inside a `var Interface{}` you must use a [Type Switch](https://golang.org/doc/effective_go.html#type_switch) a *type assertion* or *reflection* 

##To be continued...

Drafts: 
 [Type Switch Internals](Type Switch Internals.md)





