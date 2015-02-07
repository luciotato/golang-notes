#Golang concepts from an OOP point of view

##Introduction

###Objectives of this document

* Ease golang learning by associating
golang-specific *concepts* with previously known concepts in the OOP field.

* ***Promote golang usage*** by easing language understanding for people coming from a heavy OOP background

###Why github?
This is a discovery process, I'm writing this document to help myself understanding golang and maybe help others. 
This document is published in github because ***I'm expecting pull requests from people with a better understanding of golang internals.*** There are a lot of things to improve, but please, please do not start a pull request with [*"Technically...*"](http://xkcd.com/1475)

##Golang Concepts
Golang introduce words with a new golang-specific meaning, as *struct* and *interface*.  
This is not bad, but sometimes is nice to have a "translation" available to be able to understand golang-concepts by relating them to previously known concepts.

This is important in order to *understand* concepts of a new language, 
if you can *translate* a golang-word to previously known concepts, 
the learning is by far easier.

##Cheat Sheet

|Golang|Classic OOP
|----|-----|
|*struct*|class  with fields      only non-virtual methods
|*interface*|class without fields only virtual methods
|*embedding*|multiple inheritance AND composition
|*receiver*|implict *this* parameter

##Golang-struct is a class (non-virtual)

A ***golang-struct*** is a ***class*** with ***fields*** where all the methods are ***non-virtual***. e.g.:

    type Rectangle struct {
      Name          string
      Width, Height float64
    }
    func (r Rectangle) Area() float64{
      return r.Width * r.Height
    }
This can be read as (pseudo code):

    class Rectangle
      field name: string
      field Width: float64
      field Height: float64
      method Area() //non-virtual
         return this.Width * this.Height

###Constructor
  
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
    
    
Constructors example:

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

##Golang "embedded field" is ***multiple inheritance***

***multiple inheritance*** is achieved in golang by *embedding* a field ***named as other class(struct)*** inside a struct. 

Note: The official documents call it "an anonymous field", but this add to confusion, since ***the embedded field has a name***: the inherited class-name, and can be accessed by dot notation.

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
      Shape              //multiple inheritance
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
  
     
In Golang, you use class(struct)-named-fields for inheritance. So we can use the inherited class-name as field-name to access the base-class. 

Example: 

   In `var aRect Rectangle`:

 - `aRect.Name` and `aRect.NamedObj.Name` refer to the same field

 - `aRect.color` and `aRect.Shape.color` refer to the same field

###Method Shadowing

Since all ***golang-struct*** methods are ***non-virtual***, ***you cannot override methods*** (you need *interfaces* for that)

If you have a ***method show()*** for example in ***class/struct NamedObj*** and also define a ***method show()*** in ***class/struct Rectangle***,
***Rectangle_show()*** will ***SHADOW/HIDE*** the parent's class ***NamedObj_Show()***

As with base class fields, you can use the ***inherited class-name-as-field*** to access the base implementation via dot-notation, e.g.:

    var a Rectangle

    a.show()          // calls a.Rectangle_show()
    a.NamedObj.show() // calls a.NamedObj_show(), the base implementation


###Multiple inheritance and The Diamond Problem

Golang solves [the diamond problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem) by *not allowing diamonds*.

Since ***inheritance (embedded fields)*** include inherited field names in the ***inheriting class (struct)***, all *embedded* class-field-names *should not collide*. You must rename fields if there is a name collision. This rule avoids the diamond problem, by not allowing it.

Note: Golang allows you to create a "Diamond" inheritance diagram, and only will complain when you try to access a parent's class field ambiguously.

##Golang *methods* and ***"receivers" (this)***

A golang ***struct-method*** is the same as a ***class non-virtual method*** but:

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


##Structs vs Interfaces 

A ***golang-Interface*** is ***a class with no fields and ONLY VIRTUAL methods***.

The definition of *interface* in Golang is designed tom complement the *structs*. This is a very important "symbiotic" relationship in golang. *Interfaces* fits perfectly with *structs*. 

You have in Golang:
>***Structs:  *** classes, with fields, ALL NON-VIRTUAL methods
>***Interfaces:  *** classes, with NO fields, ALL VIRTUAL methods

By restricting *structs* to non-virtual methods, and  restricting *interfaces* to *all-virtual* methods and no fields. Both elements can be perfectly combined by *embedding* to create *fast* polymorphism and multiple inheritance *without the problems associated to multiple inheritance in classical OOP*

##Interfaces

A *golang-Interface* is a ***class, with NO fields, and ALL VIRTUAL methods***

Given this definition, you can use an interface to:

- Declare a var or parameter of type *interface*.
- *implement* an interface, by declaring all the *interface virtual methods* in a *concrete class (a struct)*
- *inherit(embed)* a golang-interface into another golang-interface

###Declare a var/parameter with type interface

By picturing an ***Interface*** as a ***class with no fields and only virtual abstract methods***, you can understand the advantages and limitations of *Interfaces*

By declaring a var/parameter with type *interface* you define the set of valid methods for the var/parameter. This allows you to use a form of ***polymorphism*** via ***method dispatch***

When you declare a var/parameter with type interface:

- The var/parameter has no fields
- The var/parameter has *a defined set of methods*
- When you call a method on the var/parameter, a *concrete method* is called via *method dispatch* from a jmp-table. (polymorphism via method dispatch)
- When the interface is used as a parameter type: 
  - You can call the function with *any class* implementing the interface
  - The function works for *every class* implementing the interface (the function is polymorphic)


*Note on golan implementation*: The *ITables* used for method dispatch are constructed dynamically as needed and cached. Each *class(struct)* has one ITable for each *Interface* the *class(struct) implements*, so, if all *classes(structs)* implement all interfaces, there's a *ITable* for each *class(struct)*-*Interface* combination. See: [Go Data Structures: Interfaces](http://research.swtch.com/interfaces)

####Examples from [How to use interfaces in Go](http://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go) , with commented pseudo-code

    package main
    
    import (
      "fmt"
    )
    
    /*
    class Animal
       virtual abstract Speak() string
    */
    type Animal interface {
      Speak() string
    }
    
    /*
    class Dog
      method Speak() string //non-virtual
         return "Woof!"
    */       
    type Dog struct {
    }
    func (d Dog) Speak() string {
      return "Woof!"
    }
    
    /*
    class Cat
      method Speak() string //non-virtual
         return "Meow!"
    */       
    type Cat struct {
    }
    func (c Cat) Speak() string {
      return "Meow!"
    }
    
    /*
    class Llama
      method Speak() string //non-virtual
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

By picturing an ***Interface*** as a ***class with no fields and only virtual abstract methods***, you can understand ***what is*** the golang ***Empty Interface: "Interface{}"***.

***Interface{}*** in golang is a ***class with no fields and no methods***

What can you do with a `var x Interface{}`? Well, initialy, nothing.

But, since by definition all *classes(structs)* implement ***Interface{}*** it means
that a `var x Interface{}` ***can hold any value***

***To actually use** the *value* inside a `var x Interface{}` you must use a [Type Switch](https://golang.org/doc/effective_go.html#type_switch) a *type assertion* or *reflection* 

##To be continued...

Drafts: 
 [Type Switch Internals](Type Switch Internals.md)
