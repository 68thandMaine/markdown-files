# Ruby Notes

___

## Table of Contents

| Section | Title | Section | Title |
|---|---|---|---|
| I. [Variables](#variables) |  |  |  |
| II. [Conventions](#conventions) |  |  |  |

___

Everything in Ruby is an object, including classes, and all objects belong to classes.

## Variables

There is no need for var. Ruby has four kinds of variables.

- Local Variables

    There is no need to declare variable types. That means no need for var.

**Naming Convention** :

- lowercase unless constants
- no camel case. use underscores. Like_this

To permanently change the value a variable you can use !.

```ruby
greeting = "hello"

greeting.upcase!()

"HELLO"

greeting === "HELLO"
```

Another way to do this is to declare a variable that is equal to the result of a method:

```ruby
uppercase_greeting = greeting.upcase()
```

Symbols are immutable variables that are declared with a colon. They are more efficient and take up less memory in the system.

___

## Methods In Ruby

Instead of brackets, methods in Ruby are declared with def, and closed with end.

When the Ruby interpreter sees `def` it understands that a method is about to be defined. The code betweeen `def` and `end` is known as a block.

Methods in Ruby use the keyword `self` to refer to the object a method is being called on. It's very similar to `this` in JavaScript.

Ruby methods return data in two ways:

- With an **implicit return**
  - The final statement inside the block. The method returns the output of the statement before the `end` keyword.  
- With an **explicit return**
  - Use the keyword `return` before the end of the method. This technique is useful if you want to return a value that isn't the final statement. A common usecase is with conditionals.

Methods with Arguments:

We can pass an indefinite amount of parameters to a Ruby method by using th `*` character, however Ruby is strict about arity, so we need to be careful while doing this. Arity refers to the number of arguments a method can take.

If you pass too few or too many parameters to a method, the Ruby interpreter will err. It will however tell you why you are erring.

___

## Branching

Obviously Ruby has boolean values. In true Ruby fashion, they are objects.

`nil`  is Ruby's concept of nothingness. It is considered **falsy** In Ruby `false` and `nil` are consider falsy. Everything else is **truthy**.

### if...else Statements

It is important to note that no parenthesis or braces are used in block level statements. For example:

```ruby
def old_enough(age)
  if age >= 21
    "You can drink."
  else
    "You can't drink."
  end
end
```

The age condition isn't wrapped, and the if conditional ends with the `end` keyword.

> One way to write functions that evaulate either true or false is by taking advantage of **_implicit returns_**. With an implicit return you can use self to determine the result.

If you need to create more complex conditions, we should use parenthesis as to not confuse the Ruby interpreter about order of operation.

___

## Loops

Looping in Ruby is similar to looping in JavaScript. Here are teh four main keywords that are used when creating loops:

1. `times`
2. `each`
3. `while`
4. `until`

### The 'Times` Loop

A times loop runs through something for a specified period of time. A conditional block is started with `do` and ended with `end`.

It's possible to send  arguments through a `times` loop simply by adding `|t|` to the end of the do statement:

`5.times do |t|`

### the `Each` Loop

Each loops are the most common looping technique in Ruby. They are useful for looping through collections of data and performing a function on them.

To start an each loop you eed to have a few things:

1. `.each` must be called on the collection to access the data.
2. perform some sort of action on the index Ruby is looking at.
    - `array.each do |array_element|`

### While and Until Loops

A `while` loop runs until a condition is true. An `until` loop runs until a condition becomes false. **It is important to create conditions that will be met to avoid an infinite loop**.

To avoid the infinite loop it's a best practice to declare controls outside the loop.

```ruby
x = 0
array = []
while(x<10)
    x = x + 1
    array.push(x)
end
```

Until is more or less the same thing. It runs until a condition is met.

## Scope in Ruby

Scope gates determine when variables fall out of scope. They are:

- `module`
- `class`
- `def`
- `end`

There is something called *flat scope* that allows for using variables outside of their scope.

### Variables

The concept of variables applies across programming languages, and Ruby is no different. There are four types of variables:

- Global
  - Available everywhere in a Ruby program
  - Always preceded by the `$` symbol
- Local
- Class
  - Available everywhere inside a class
  - Preceded by the `@@` symbol
- Instance
  - Available to a single instance of an object
  - Preceded by the `@` symbol

___

## Ruby Gems

Ruby packages are called gems. Each application that uses gems will need a file to manage the project's gems. Name it `Gemfile` and place it in the root directory.

The `Gemfile` has no extension and is always capitalized.

To install Gems to a project run `bundle install` in the root of the project. Anytime the `Gemfile` is altered, run `bundle` in the terminal to update the project.

Run `bundle update` to update all gems, and `bundle update <gem-name>` to update specific gems. It is a good idea to update gems one at a time to make sure the project still works and to target which gem introduces breaking changes to the project.

___

## Instantiating New Objects and Creating Custom Classes

By adding `.new()` to the end of a type, we can create new variables of the type rather than using literal notation.

Not all classes can be instantiated with literal notation so it is best to create new objects with this keyword.

Classes always start with an uppercase letter and are written with PascalCase.

In Ruby all classes recognize a method called  `initialize`. Any code within `initialize` will run as soon as an object is created.

When creating classes we need to use **instance variables** so that new objects can access different methods within the class. **Instance variables are preceeded with `@`**.

By creating instance variables, we are able to start instantiating the various properties of an object.

```ruby

class Cat
  def initialize(name, age, colors)
    @name = name
    @age = age
    @colors = colors
  end
end
```

To access the values of the instance variables we need to have **reader methods**. Reader methods seem to be very similar to getters.

```ruby
def name
  @name
end
```

Reader methods belong in the class, so the whole code so far looks like this:

```ruby
class Cat
  def initialize(name)
    @name = name
  end

  def name
    @name
  end
end
```

The `name` method is considered an instance method because it is called on an instance of the class. As a programmer, if you want to interact with instance variables, you just need to write the`@variable_name` into the method block and mutate it however you choose.

___

## The Hash Class

Elements in a hash are stored as key-value pairs.

### Important Methods in the Hash Class

1. `store()`

    - This method adds elements to a hash. The first argument to this method is for the key, and the second argument to the method is the value.
  
2. `fetch()`

    - This method  takes a key as an argument and returns its value.

A more popular way for creating hashes is by using litertal notation. Note how the following example does not use the `store()` method:

```ruby
numbers = {"michael" => 5039945979, "jessica" => 1232311242, "daniel" => 9172339078 }

numbers.fetch("daniel")
=>9172339078
```

Here are some good hash methods:

| Method | Return |
|---|---|
| `include?(key)` | returns true if the key is present in the hash |
| `invert()` | returns a new hash using the values as keys, and they keys as values |
| `key(value)` | returns the key corresponding to the given value |
| `keys()` | returns an array of all the keys |
| `length()` | returns the number of entries |
| `merge(other_hash)` | combines two hashes into one |

___


___

## Conventions

- no semi colons to end statements
- no camel casing. use underscores and lower case.
  - `like_this`
- no padding with brackets or parenthesis
- no extra lines in blocks
- Use the implicit `self`
- no comma at the end of an array or hash.
- methods that return a boolean should end with a question mark
- methods that modify the receiver or `self` should end with an exclamation mark

### Project File Structure

    ruby_project
    |__lib
      |_ruby_logic.rb
    |__spec
      |__ruby_logic_spec.rb
    |__Gemfile
    |__ruby_script.rb <--- This is optional and only necessary if you plan to have a script.
    |__README.md

The `ruby_script` file is only necessary if you plan to run the file as a script in the terminal. Files in `lib` and `spec` should never have a shebang. Shebangs are this `#!/usr/bin/ruby`. As for the `ruby_script`, the first line will be the shebang, and the second line is generally a require statement that uses a relative path from the current directory to the file we need.
