The ooc runtime
===============

Brief overview
--------------

This document is a description of the ooc runtime and the C translation of every single construct in ooc.  
It will probably be the biggest note in this repository and it may get pretty chaotic but I will try to structure it as well as I can.  

Types
-----

### Classes

Classes are translated into two separate C structures.  

One is the object class itself, containing its fields as well as a super object field.  
This super object field is the first field of the struct and has the type of the object class struct of the superclass, if any.  

The other structure is the metaclass structures.  
It is basically an object class struct for the Class type, that contains some information on the class (like the name and instance size).  
In addition to this information, this struct also serves the role of a vtable, as it contains the function pointers of the virtual functions of the class.  
Finally, it contains static fields of the class.  
Unlike the object class struct, there is no superclass first field (although there is a superclass field defines in Class itself)  
The layout of the struct is identical to its superclasses' (up to the point where new methods are added).  
This way, casting it to its superclasse's struct type gives us access to the overwritten methods.  

All classes extend Object by default.  

### Compound covers

Compound covers are similar to classes.  
The difference is that they don't extend Object by default and that they are pass by value by default.  
They can extend classes or other covers.  
Extending a class is a bit tricky, because the methods defined by it are pass by reference, so they should be extended using th func@ keyword but the compiler should take care of telling us that.  

### Template covers

Template covers are just compoud covers with an extra resolution and name mangling phase, so everything said above still holds true.  

### Covers from C types

Covers from C types only generate a metaclass struct.  
The value of a cover from instead becomes of the C type we are wrapping.  
This means that they are definitely not of type Object and cannot be used like they are.  
I'm not sure if I should implement instanceOf? and co. like rock does, by automatically replacing it with calls on the metaclass but I am leaning against it.  
This would only give the illusion that everything is an Object, when in fact it is not.  

### Enums

Enums translate just like covers from Int with static This fields.  

### Interfaces

Interfaces are perhaps the trickiest thing to implement when compiling down to C.  
The basic idea behind the implementation is that an interface is just a vtable that must be filled for every class that implements it.  
This means that implementing an interface generates an additional vtable pointing to functions inside the metaclass of the implementor.  
Static methods defined by the interface are also added into the vtable.  

In detail, the following happen:  

An interface metaclass struct is created.  
This metaclass struct isa just the same as a class, pointing to the interface's method stubs that basically take an interface value struct and use the metaclass to call a method on the data.  

An interface vtable struct is created.  
The first argument (this argumnet) of the interface methods is defined as a void pointer iinside this struct.  
An interface value struct is created, which contains a void (or byte, whatever) pointer to some data (the object that implements the interface) and a pointer to an interface vtable.  
When a type implements an interface, it creates a new (static) instance of the interface vtable, pointing to functions it needs to implement.  

Anytime we pass some value to a function that expects an interface value, it is wrapped into an interface value struct.  

This is the system!  
This actually allows types to implement interfaces after definition, which is not currently supported syntax in rock but will be added in my implementation.  
This will be very useful to implement user defined interfaces on sdk types and have them behave well in the library.  
I can think of many applications, for example serialization.  

The only downside is the fact that you cannot implement a method byref in a cover.  
This is because of the fact that the cover is wrapped into an interface value struct and the vtable methods are called directly on it.  
There is no way to specify if the method should be called on the reference itself or by value, unless we create an extra layer of stubs that take pointers to covers and call the cover funcs.  
That way, the data of the interface struct could point to a pointer of the cover.  
I will actually consider this :P
