The ooc typesystem
==================

Brief overview
--------------

In ooc, there are no builtin types, everything is defined, at the lowest level in terms of C types.  
There are three different type definition constructs: enum, cover and class, as well as interfaces.  
In addition to defining a type, Metaclasses are generated, which are themselves objects of type Class (which is defined as a class)  
Enum is the simplest construct. Think of them like C enums (Int backed), with the extra ability of having methods.  
Enums can also take a custom increment and custom values for their enumerated values.  
Enums can be defined as extern, to wrap C int values into an ooc enum.  

Classes are what you would expect them to be. They support single inheritance and can be defined as abstract.  
All class types inherit from Object and are passed by reference and allocated on the heap (with the default new functions).  

Covers are the most complicated types in the ooc typesystem. They can also extend classes (or other covers).  
There are three types of covers: covers from C types, template covers and compound covers.  

Covers from C types wrap around a C type as you would expect and can define extra functions.  
They are mainly used in C library bindings, to make them ooc-like, as many C libraries are written in an OO fashion.  
They can be value or reference types depending on the C type they wrap.  
You can explicitely mark a cover from a C type as byref (in the event, say, that you cover a typedef'ed C pointer type). // Foo: cover from c_foo(byref) {}  

Template covers and compound covers are similar.  
They are pass by value types, so their methods need to be handled careful.  
Template covers are a simple template system you can use on top of covers.  

// TODO: template funcs, classes?  
// TODO: add bit about addons  

Generics
--------

In ooc, generics are reified. That means that, instead of replacing generic types at compile-time, the meta-class object is passed to the genefic type or function we use.  
Then, we can do runtime checks on the meta-class and cast the generic value if needed.  

In addition to "simple" generics, generic constraints are supported.  
Generic constraints are ways of limiting the generic types passed to be compatible with another type.  
For example, this: &lt;T, V, K: HashMap&lt;T, V&gt;&gt; will guarantee K is a subtype of HashMap with equal generic parameters T, V as the original construct (type/function).  

Interfaces
----------

// TODO: should covers and enums really be able to implement interfaces?
// How is that translated into the runtime?

An interface is a special declaration.  
It doesn't define a type per se, or we could rather say it defines an incomplete type.  
An interface can only define functions as well as virtual properties and operators, which are essentially functions. // TODO: functions and operators without body, what about properties?  
Any ooc type declaration, including another interface declaration, can implement an interface and thus override every function defined.  
Interface declarations can be generic, in which case they must be implemented with a generic type of the type that implements them. // TODO: template-interfaces where you can implement for a specific type (only)?  

// TODO: ad-hoc/after the fact/non-declaration implementation? (A-la rust traits)  

Pointer types
-------------

There are a couple of pointer types in ooc.  
Those are raw pointers (sometimes also referred to as C arrays), with the T* syntax and references, with the T@ syntax.  
References are auto-dereferenced when accessed (and generally used as if they were of the type they wrap) and are translated to raw pointers.  
However, a reference must explicitely be defined by taking the address of the value they reference to via the & operator (same as raw pointers)  
For example, a: Int@ must take a value like 42&, not 42.  

Function types
--------------



Scoring system
--------------

The ooc scoring system is arguably the most important part of the ooc typesystem alongside common root computation.  
It is used to find the most compatible function declaration to match a call, among other things.  

Given two types and a maximum score possible, we can define a score of those two types following a plethora of criteria.  
A score of -1 is convention for "something not resolved yet" (which shouldn't happen in oc-revival) while another negative value means the two types are incompatible.  

// TODO: check this, add more cases if needed  
The list of those criteria is the following, in descending order of importance (where a base type is a class, cover or enum type):  

- Two equal types have a score of MAX
- An enum type and a decimal type have a score of MAX
- An enum type and a floating type have a score of 3MAX/4
- Two enum types have a score of 3MAX/4
- Two decimal types have a score of MAX
- Two floating point types have a score of MAX  
- A decimal and a floating point type have a score of 3MAX/4
- Two class types have a score of MAX/(1 + inheritance distance) if one inherits the other
- Two function types are equal if all their argument types and their return types are equal
- Two pointer types have a score equal to the score of their inner types
- An interface type and a base type have a score of MAX if the base type provides an implementation of that interface declaration
- A generic type and a base type have a score of 3MAX/4
- A pointer type and "Pointer" have a score of MAX/2
- "Pointer" and other byref types have a score of MAX/4
- All remaining cases have a score of -MAX

// TODO: More advanced scoring for covers and pointers to covers?  
// E.g.: Foo: cover from foo {}; Bar: cover from foo* {}; <- detect Foo* is fully compatible with Bar  

Built-in coercion
-----------------

In some specific cases, built-in coercion can happen.  
Here is a list of places where this can happen:  
- Types compatible to "Pointer" can be coerced to "Bool" in condition expressions or logical operators
- An access to a global function (of type "Pointer") can be coerced to any function type
- An ooc array can be coerced to a C array (raw pointer)
- // TODO: add more?

Common roots
------------
