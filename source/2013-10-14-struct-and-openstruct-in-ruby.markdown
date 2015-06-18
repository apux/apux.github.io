---
title: Struct and OpenStruct in Ruby
date: 2013-10-14
tags: [Ruby, Struct, OpenStruct]
---

The Ruby's `Struct` and `OpenStruct` classes are very useful for some tasks.
They act like a simple data structure, similar to hash, but they also allow to
access their attributes by calling them directly from the object and not only
using the brackets.

# Struct

Let's see and example with `Struct`.

```ruby
Computer = Struct.new :ram, :hard_disk
computer = Computer.new
computer[:ram] = "4 MB"
computer.hard_disk = "500 GB"
 
# Access data with []
computer[:ram]       # => "4 MB"
computer[:hard_disk] # => "500 GB"
 
# Access data as attributes
computer.ram       # => "4 MB"
computer.hard_disk # => "500 GB"
```

In line 1, we defined a struct indicating the attributes it will contain. This
instruction is very interesting: in theory, it should return a `Struct` object,
but it doesn't. Actually, it returns a `Class` object, the same type of object
that Ruby generates when a class is defined (yes, in Ruby, the class definition
is also an object, but we will not go into details for now). This object can be
assigned to a variable (or a constant, like in this case), and it is also
possible to generate objects from it, as we do in line 2. 

In line 3, we assign a value to one of the attributes using the hash syntax,
and in the line 4 we do the same with the other attribute but using the method
syntax. The lines 7 and 8, and the lines 11 and 12 show that the data can be
accessed in both ways too.

Also, `Struct` allows to define methods as part of the structure. Let's see:

```ruby
Computer = Struct.new(:ram, :hard_disk) do
  def descripcion
    "Computer with #{ram} ram and #{hard_disk} hard disk."
  end
end
 
computer = Computer.new("4 MB", "500 GB")
computer.descripcion
# => Computer with 4 MB ram and 500 GB hard disk.
```

In this example we can see how a method is defined in a `Struct` and also we
see that it is possible to create the object by indicating the values it will
contain, like if we defined an `initialize` method that receives these
parameters. The order of the values must match the order of the parameters
defined for the structure (see lines 1 and 7).

The params of `Struct` are not strict. If we declare that it receives 3 params
and when creating we only send 1, the remaining 2 are initialized with nil. On
the other hand, if we send more than 3 params, we get an exception.

```ruby
computer = Struct.new(:ram, :hard_disk, :processor).new("4 MB")
# => #<struct ram="4 MB", hard_disk=nil, processor=nil>

computer = Struct.new(:ram).new("4 MB", "500 GB", "2.5 GHz", "1024x768")
# => ArgumentError: struct size differs
```

In the previous examples we can also see that it is possible to create
`Struct`'s objects in a line.

As we can see, it is a very versatile class that allows us to create objects
with certain behavior with no need of creating a class for that.

# OpenStruct

`OpenStruct` behaves similarly, but it allows us to add attributes on the fly,
ie with no need to define them previously. Let's see the following example:

```ruby
require 'ostruct'
 
computer = OpenStruct.new ram: '4 MB', hard_disk: '500 GB'
computer.ram # => '4 MB'
computer[:hard_disk] # => "500 GB"
 
# attributes can be created 'on the fly'
computer.screen = "1024x768"
computer.screen   # => "1024x768"
computer[:screen] # => "1024x768"
```

The first thing to note is that now we need to include the `ostruct` library
because it is not part of the `core` but of `stlib`; this is done in line 1. In
line 3 an `OpenStruct` object is created and initialized at the same time.
Unlike `Struct`, there is no intermediate class, but the created object
directly. To initialize it we can assign a hash directly, like in the example.
The lines 4 and 5 verify the values can be accessed the same way they can be
accessd with `Struct`.

In line 8 we assign a value to a no-defined attribute. We didn't indicate our
structure to accept values for `screen`, but `OpenStruct` knows how to handle
these cases by creating this attribute on the fly. In lines 9 and 10 we see it
is possible to access it with no problem.

Another important difference is that `OpenStruct` does not allow to define
methods in its declaration, as `Struct` does.

# Conclusions

The classes `Struct` and `OpenStruct` provide a simple way to create data
structures with the behavior of a class. This is particulary useful when
refactoring and we are not sure about what new classes we need to create, then
we can use `Struct` or `OpenStruct` as an intermediate step. Obviously, if the
structure becomes more complex and requires more effort to be mantained, it is
time to move it to a real class.

`OpenStruct`, besides not being part of the `core`, is slower than `Struct`,
thus we must prefer `Struct` whenever possible and use `OpenStruct` only when
necessary.

Here are the docs for [Struct](http://www.ruby-doc.org/core-2.0/Struct.html)
and
[OpenStruct](http://www.ruby-doc.org/stdlib-2.0/libdoc/ostruct/rdoc/OpenStruct.html).
