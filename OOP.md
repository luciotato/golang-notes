# Golang concepts from an OOP point of view

## Introduction

### Objectives of this document

* Ease golang learning by associating golang-specific *concepts* with previously known concepts in the OOP field.

* ***Promote golang usage*** by easing language understanding for people coming from a heavy OOP background

### Why github?
This is a discovery process, I'm writing this document to help myself understand golang and maybe help others.
This document is published in github. **_Pull requests are welcomed_**. There are a lot of things to improve, but please, please do not start a pull request with [*"Technically...*"](http://xkcd.com/1475)

## Golang Concepts
Golang introduces words with a new golang-specific meaning, such as *struct* and *interface*.
This is not bad, but sometimes it is nice to have a "translation" available to be able to understand golang-concepts by relating them to previously known concepts.

This is important in order to *understand* concepts of a new language.
If you can *translate* a golang-word to previously known concepts,
the learning is by far easier.

## Cheat Sheet

|Golang|Classic OOP
|----|-----|
|*struct*|class  with fields, only non-virtual methods
|*interface*|class without fields, only virtual methods
|*embedding*|multiple inheritance AND composition
|*receiver*|implicit *this* parameter

## Golang-struct is a class (non-virtual)

A ***golang-struct*** is a ***class*** with ***fields*** where all the methods are ***non-virtual***. e.g.:

```go
type Rectangle struct {
	Name          string
	Width, Height float64
}

func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}
```

This can be read as (pseudo code):

```go
class Rectangle
  field Name: string
  field Width: float64
  field Height: float64
  method Area() //non-virtual
     return this.Width * this.Height
```

### Constructor

- There is a *zero-value* defined for each core-type, so if you do not provide values at instantiation, all fields will have the zero-value

- No specific in-class constructors. There is a *generic constructor* for any class instance, receiving a generic literal in a JSON-like format and using reflection to create instances of any class.

pseudo-code for the generic constructor:

```go
function construct ( class,  literal)

  helper function assignFields ( object, literal)  //recursive
    if type-of object is "object"
        if literal has field-names
            for each field in object.fields
                if literal has-field field.name
                    assignFields(field.value,  literal.fields[field.name].value) //recurse
        else
        //literal without names, assign by position
            for n=0 to object.fields.length
                assignFields(object.fields[n].value,  literal.fields[n].value) //recurse
    else
        //atom type
        set object.value = literal.value


// generic constructor main body
var classInstance = new class
assignFields(classInstance, literal)
return classInstance
```

Constructors example:

```go
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
```

Output:

```
{ 0 0}
{I'm b. 10 20}
{ 14 12}
```


## Golang "embedding" is akin to ***multiple inheritance with non-virtual methods***

By *embedding* a struct into another you have a mechanism similar to ***multiple inheritance with non-virtual members***.

Let's call "base" the struct embedded and "derived" the struct doing the embedding.

After embedding, the base fields and methods are directly available in the derived struct. Internally a *hidden field* is created, named as the base-struct-name.

Note: The official documents call it "an anonymous field", but this add to confusion, since ***internally there is an embedded field named as the embedded struct***.

Base fields and methods are directly available as if they were declared in the derived struct, but *base fields and methods* **can be "shadowed"**

**Shadowing** means defining another field or method *with the same name (and signature)* of a *base* field or method.

Once shadowed, the only way to access the base member is to use the *hidden field* named as the base-struct-name.

```go
type base struct {
	a string
	b int
}

type derived struct {
	base // embedding
	d    int
	a    float32 //-SHADOWS
}

func main() {
	var x derived

	fmt.Printf("%T\n", x.a) //=> x.a, float32 (derived.a shadows base.a)

	fmt.Printf("%T\n", x.base.a) //=> x.base.a, string (accessing shadowed member)
}
```

All base members can be accessed via the *hidden field* named as the base-struct-name.

It is important to note that all *inherited* methods **are called on the hidden-field-struct**. It means that a base method cannot see or know about derived methods or fields. Everything is non-virtual.

**When working with structs and embedding, everything is STATICALLY LINKED. All references are resolved at compile time.**

### Multiple embedding

```go
type NamedObj struct {
	Name string
}

type Shape struct {
	NamedObj  //inheritance
	color     int32
	isRegular bool
}

type Point struct {
	x, y float64
}

type Rectangle struct {
	NamedObj            //multiple inheritance
	Shape               //^^
	center        Point //standard composition
	Width, Height float64
}

func main() {
	var aRect = Rectangle{
		NamedObj{"name1"},
		Shape{NamedObj{"name2"}, 0, true},
		Point{0, 0},
		20, 2.5
	}

	fmt.Println(aRect.Name)
	fmt.Println(aRect.Shape)
	fmt.Println(aRect.Shape.Name)
}
```

This can be read: (pseudo code)

```go
class NamedObj
   field Name: string

class Shape
   inherits NamedObj
   field color: int32
   field isRegular: bool

class Rectangle
   inherits NamedObj
   inherits Shape
   field center: Point
   field Width: float64
   field Height: float64
```

Example:

   In `var aRect Rectangle`:

 - `aRect.Name` and `aRect.NamedObj.Name` refer to the same field

 - `aRect.color` and `aRect.Shape.color` refer to the same field

 - `aRect.name` and `aRect.NamedObj.name` refer to the same field, but `aRect.NamedObj.name` and `aRect.Shape.NamedObj.name` are **different fields**

 - `aRect.NamedObj` and `aRect.Shape.NamedObj` are the same type, **but refer to different objects**


### Method Shadowing

Since all ***golang-struct*** methods are ***non-virtual***, ***you cannot override methods*** (you need *interfaces* for that)

If you have a ***method show()*** for example in ***class/struct NamedObj*** and also define a ***method show()*** in ***class/struct Rectangle***,
***Rectangle/show()*** will ***SHADOW*** the parent's class ***NamedObj/Show()***

As with base class fields, you can use the ***inherited class-name-as-field*** to access the base implementation via dot-notation, e.g.:

```go
type base struct {
	a string
	b int
}

//method xyz
func (this base) xyz() {
	fmt.Println("xyz, a is:", this.a)
}

//method display
func (this base) display() {
	fmt.Println("base, a is:", this.a)
}

type derived struct {
	base // embedding
	d    int
	a    float32 //-SHADOWED
}

//method display -SHADOWED
func (this derived) display() {
	fmt.Println("derived a is:", this.a)
}

func main() {
	var a derived = derived{base{"base-a", 10}, 20, 2.5}

	a.display() // calls Derived/display(a)
	// => "derived, a is: 2.5"

	a.base.display() // calls Base/display(a.base), the base implementation
	// => "base, a is: base-a"

	a.xyz() // "xyz" was not shadowed, calls Base/xyz(a.base)
	// => "xyz, a is: base-a"
}
```

### Multiple inheritance and The Diamond Problem

Golang solves [the diamond problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem) by *not allowing diamonds*.

Since ***inheritance (embedded fields)*** include inherited field names in the **_inheriting
class (struct)_**, all *embedded* class-field-names *should not collide*. You must rename fields if there is a name collision. This rule avoids the diamond problem, by not allowing it.

Note: Golang allows you to create a "Diamond" inheritance diagram, and only will complain when you try to access a parent's class field ambiguously.

## Golang *methods* and ***"receivers" (this)***

A golang ***struct-method*** is the same as a ***class non-virtual method*** but:

- It is defined *outside* of the ***class(struct)*** body
- Since it is outside the class, it has an *extra section* before the method name to define the **_"receiver" (this)_**.
- The extra section defines ***this*** as an ***explicit parameter*** (The ***this/self*** parameter is implicit in most OOP languages).
- Since there is such a special section to define **_this (receiver)_**, you can also select a ***name*** for ***this/self***. Idiomatic golang is to use a short var name with the class initials. e.g.:

Example

```go
//class NamedObj
type NamedObj struct {
	Name string
}

//method show
func (n NamedObj) show() {
	Println(n.Name) // "n" is "this"
}

//class Rectangle
type Rectangle struct {
	NamedObj      //inheritance
	Width, Height float64
}

//override method show
func (r Rectangle) show() {
	Println("Rectangle ", r.Name) // "r" is "this"
}
```

  Pseudo-code:

```go
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
```

Using it:

```go
func main() {
	var a = NamedObj{"Joe"}
	var b = Rectangle{NamedObj{"Richard"}, 10, 20}

	a.show("Hello")
	b.show("Hello")
}
```

Output:

```
Hello I'm Joe
Hello I'm Richard
- I'm a Rectangle named Richard
```

## Structs vs Interfaces

A ***golang-Interface*** is ***a class with no fields and ONLY VIRTUAL methods***.

The *interface* in Golang is designed to complement *structs*. This is a very important "symbiotic" relationship in golang. *Interface* fits perfectly with *structs*.

You have in Golang:
>**_Structs:_** classes, with fields, ALL NON-VIRTUAL methods

>**_Interfaces:_** classes, with NO fields, ALL VIRTUAL methods

By restricting *structs* to non-virtual methods, and  restricting *interfaces* to *all-virtual* methods and no fields. Both elements can be perfectly combined by *embedding* to create *fast* polymorphism and multiple inheritance *without the problems associated to multiple inheritance in classical OOP*

## Interfaces

A *golang-Interface* is a ***class, with NO fields, and ALL VIRTUAL methods***

Given this definition, you can use an interface to:

- Declare a var or parameter of type *interface*.
- *implement* an interface, by declaring all the *interface virtual methods* in a *concrete class (a struct)*
- *inherit(embed)* a golang-interface into another golang-interface

### Declare a var/parameter with type interface

By picturing an ***Interface*** as a ***class with no fields and only virtual abstract methods***, you can understand the advantages and limitations of *Interfaces*

By declaring a var/parameter with type *interface* you define the set of valid methods for the var/parameter. This allows you to use a form of ***polymorphism*** via ***method dispatch***

When you declare a var/parameter with type interface:

- The var/parameter has no fields
- The var/parameter has *a defined set of methods*
- When you call a method on the var/parameter, a *concrete method* is called via *method dispatch* from a jmp-table. (polymorphism via method dispatch)
- When the interface is used as a parameter type:
  - You can call the function with *any class* implementing the interface
  - The function works for *every class* implementing the interface (the function is polymorphic)


*Note on golang implementation*: The *ITables* used for method dispatch are constructed dynamically as needed and cached. Each *class(struct)* has one ITable for each *Interface* the *class(struct) implements*, so, if all *classes(structs)* implement all interfaces, there's a *ITable* for each *class(struct)*-*Interface* combination. See: [Go Data Structures: Interfaces](http://research.swtch.com/interfaces)

#### Examples from [How to use interfaces in Go](http://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go) , with commented pseudo-code

```go
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
     return "LaLLamaQueLLama!"
*/
type Llama struct {
}

func (l Llama) Speak() string {
	return "LaLLamaQueLLama!"
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
```

### The empty Interface

By picturing an ***Interface*** as a ***class with no fields and only virtual abstract methods***, you can understand ***what*** the golang ***Empty Interface: "Interface{}"*** is.

***Interface{}*** in golang is a ***class with no fields and no methods***

But, since by definition all *classes(structs)* implement ***Interface{}*** it means
that a `var x Interface{}` ***can hold any value***

What can you do with a `var x Interface{}`? Well, initially, nothing, because you don't know the type of the concrete value stored inside the `var x Interface{}`

***To actually use*** the *value* inside a `var x Interface{}` you must use a [Type Switch](https://golang.org/doc/effective_go.html#type_switch), a *type assertion*, or *reflection*

There is no automatic type-conversion **from** `Interface{}`

## Using struct embedding

When you use ***only struct embedding*** to create a multi-root hierarchy, via multiple inheritance, you must remember that *all struct methods are non-virtual*.

That's why a *struct* hierarchy ***is always faster***. When no interfaces are involved, the compiler ***knows*** exactly what concrete function to call, so all calls are direct calls resolved at compile time. Also the functions can be inlined.

**_The problem is with [second-level methods][1]_**. You cannot alter the execution of a base-class second-level method, because all struct-methods are non-virtual.
You will inherit fields and methods, but methods are always executed on the base-class context.

**When working with structs and embedding, everything is STATICALLY LINKED. All references are resolved at compile time. There are no virtual methods in structs**

## Using interfaces

When you use ***only interface embedding*** to create a multi-root hierarchy, via multiple inheritance, you must remember that *all interface methods are virtual*.

That's why a *interface* hierarchy ***is always slower than structs***. When interfaces are involved, the compiler ***does not know*** at compile time, what concrete function to call. It depends on the concrete content of the interface var/parameter, so all calls are ***resolved at run-time via method dispatch***. The mechanism is fast, but not as fast as compile-time resolved concrete calls. Also, with an interface-method call, since the concrete function to call is unknown until run-time, the call cannot be inlined.

**_The advantage of Interfaces is that: with [second-level methods][1]_**. You can alter the execution of the base-interface second-level method, because all interface-methods are virtual.
Since all calls are made via method-dispatch, methods are executed on the context of the **actual instance**.


## To be continued...

Drafts:
 [Type Switch Internals](Type%20Switch%20Internals.md)

[1]: second-level%20methods.md
