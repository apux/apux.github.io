---
title: Super in Ruby
date: 2014-01-13
tags: [Ruby]
---

In object oriented programming languages, inheritance is a great concept to use. It is not just about code reuse, but about design. However, we can reuse code when using inheritance, but we need to know how it works within our language and how classes and methods interact in order to take benefit of it.

## Methods in subclasses and superclasses

When executing an object's method in Ruby, the interpreter search for this method in the object's class definition, if it is found, then it is executed, otherwise it search in the object's class's superclass, and so on. Example:

```ruby
class Person
  def sleep
    puts "Sleeping..."
  end

  def eat
    puts "Eating..."
  end

  def run
    puts "Running..."
  end
end

class Employee < Person
  def work
    puts "Working..."
  end
end
```

Executing:

```ruby
eployee = Employee.new
employee.work
# Working...
employee.sleep
# Sleeping...
```

Both methods were executed, but `work` was taken from the `Employee` class, while `sleep` was taken form the `Person` class.

When a method is overridden, the superclass's method is hidden and the subclass's one is the only one that can be executed.

```ruby
class Person
  def sleep
    puts "Sleeping..."
  end
   
  def eat
    puts "Eating..."
  end
   
  def run
    puts "Running..."
  end
end
 
class Employee < Person
  def work
    puts "Working..."
  end
 
  def sleep
    puts 'sleeping at work.'
  end
end
```

Here, the `work` method is defined in the `Person` class, but it is also defined in the `Employee` class, so when is executed from the `employee` object, the one executed is in the `Employee` class.

```ruby
employee = Employee.new
employee.sleep
# sleeping at work.
```

## Calling a superclass's method from a subclass

It is possible that the `Employee`'s `sleep` method needs to reuse code from `Person`'s `sleep`. Instead of copy-paste, we can simply call the superclass's method. But there is a gotcha: we can not call `sleep` directly because that would provokes a recursive call over itself. Example:

```ruby
class Employee < Person
  def work
    puts "Working..."
  end
 
  def sleep
    sleep # try to call the superclass's method
    puts "at work"
  end
end
```

Trying execute the `sleep` method will generate a `stack level too deep (SystemStackError)`. In these cases, we must use the keyword `super`.

```ruby
class Employee < Person
  def work
    puts "Working..."
  end
 
  def sleep
    super # try to call the superclass's method
    puts "at work"
  end
end
```

Thus, the `Employee`'s `sleep` method can use the functionality of the `Person`'s `sleep` method.

```ruby
employee = Employee.new
employee.sleep
# Sleeping...
# at work
```

## How to call `super` with parameters

Now, this method has a particular behavior that is necessary to know in order to avoid some mistakes.

If the superclass's method receives parameters, is possible to pass them explicitly or implicitly. Explicitly means we have to specify the parameters during the calling; implicitly means we can just call super with no parameters. In the following example we can see both behaviors. `slepp` and `play` receives two parameters, both in superclass as in subclass. However, calling super in the first method is made with parameters and in the second method is made without parameters. Both works correctly.

```ruby
class Person
  def sleep(hours, minutes)
    puts "Sleeping... #{hours} hours, #{minutes} minutes"
  end
 
  def eat
    puts "Eating..."
  end
 
  def run
    puts "Running..."
  end
 
  def play(hours, minutes)
    puts "Playing... #{hours} ours, #{minutes} minutes"
  end
end
 
class Employee < Person
  def work
    puts "Working..."
  end
 
  def sleep(hours, minutes)
    super(hours, minutes)
    puts "... at work"
  end
 
  def play(hours, minutes)
    super
    puts "... at work"
  end
end
```

Executing, we have this result:

```ruby
employee = Employee.new
employee.sleep(2, 30) 
# Sleeping... 2 hours, 30 minutes
# ... at work
employee.play(3, 10) 
# Playing... 3 hours, 10 minutes
# ... at work
```

Calling `super` with no parameters provokes ruby to include them automatically.

### Different number of parameters

Ideally, we should follow the [Liskov substitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle) and therefor the scenario we are going to see would never happen. However, in the real word, it can happen (sometimes, more often than we want), so we have to contemplate it. If the superclass's method receives a different number of parameters, an error will be raised.

```ruby
class Person
 
  # ...
 
  def rest(hours, minutes)
    puts "Resting... #{hours} hours, #{minutes} minutes"
  end
end
 
class Employee < Person
 
  # ...
 
  def rest(minutes)
    super
    puts "... at work"
  end
end
```

Executing:

```ruby
employee = Employee.new
employee.rest(90)
# ArgumentError: wrong number of arguments (1 for 2)
```

The problem was Ruby detected the call to `super` with no parameters and, automatically, adds them. The problem was that the number of parameters doesn't match. In these cases, we have to call `super` specifying the parameters explicitly.

```ruby
class Person
 
  # ...
 
  def rest(hours, minutes)
    puts "Resting... #{hours} hours, #{minutes} minutes"
  end
end
 
class Employee < Person
 
  # ...
 
  def rest(minutes)
    super(minutes / 60, minutes % 60)
    puts "... at work"
  end
end
```

Execution:

```ruby
employee = Employee.new
employee.rest(90)
# Resting... 1 hours, 30 minutes
# ... at work
```

### Superclass's method with no parameters and subclass's methods with parameters

One more scenario. What happen when the superclass's method doesn't receive parameters but the subclass's one does?

```ruby
class Person
 
  # ...
 
  def program
    puts "Programming..."
  end
end
 
class Employee < Person
 
  # ...
 
  def program(project)
    super
    puts "... project #{project}"
  end
end
```

We get the following error:

```ruby
employee = Employee.new
employee.program("Personal")
# person.rb:5:in `program': wrong number of arguments (1 for 0) (ArgumentError)
#        from person.rb:15:in `program'
```

The error is that when executing `super`, Ruby adds the parameters automatically and that provokes the fail because the superclass's method doesn't receive any parameters. For these cases, the solution is to call the method with the parenthesis, like if we were including parameters, but leaving them empty. This way, Ruby understand that it should not add parameters automatically.

```ruby
class Person
 
  # ...
 
  def program
    puts "Programming..."
  end
end
 
class Employee < Person
 
  # ...
 
  def program(project)
    super()
    puts "... project #{project}"
  end
end
```

Execution:

```ruby
employee = Employee.new
employee.program("Personal")
# Programming...
# ... project Important
```

### The `initialize` method

The same principle applies for the constructor.

```ruby
class Person
 
  def initialize
    # code to initialize a person
  end
 
  # ...
 
end
 
class Employee < Person
 
  attr_reader :employee_number
 
  def initialize(employee_number)
    super()
    @employee_number = employee_number
  end
 
  # ...
 
end
```

## Final thoughts

Using inheritance in Ruby implies knowing how the methods interact inside the hierarchy, specially when we want to reuse the behaviour it provides. Using `super` is essential in these cases, and knowing how it works allows us to use the language's properties in a correct way.
