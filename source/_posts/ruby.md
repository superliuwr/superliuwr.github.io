---
title: Ruby
date: 2018-06-24 20:45:30
categories:
- RoR
tags:
- Ruby
---
# Tools

<!-- more -->

* irb
* rvm
  * rvm list
  * rvm list known
  * rvm install 2.4.2
  * rvm remove 1.9.2
  * rvm 2.0.0 --default
* gem
  * RubyGems is a package manager for the Ruby programming language that provides a standard format for distributing Ruby programs and libraries (in a self-contained format called a "gem"), a tool designed to easily manage the installation of gems, and a server for distributing them. 
* bundler
  * a bunch of gems defined in a file
  * http://bundler.io/

# Syntax
``` ruby
# This is a comment

# 会在程序运行之前被调用
BEGIN {
   puts "初始化 Ruby 程序"
}
# 会在程序的结尾被调用
END {
   puts "停止 Ruby 程序"
}

# In Ruby, (almost) everything is an object.
# This includes numbers…
3.class #=> Integer

# …strings…
"Hello".class #=> String

# …even methods!
"Hello".method(:class).class #=> Method

# Some basic arithmetic
1 + 1 #=> 2
8 - 1 #=> 7
10 * 2 #=> 20
35 / 5 #=> 7
2 ** 5 #=> 32
5 % 3 #=> 2

# Bitwise operators
3 & 5 #=> 1
3 | 5 #=> 7
3 ^ 5 #=> 6

# Arithmetic is just syntactic sugar
# for calling a method on an object
1.+(3) #=> 4
10.* 5 #=> 50
100.methods.include?(:/) #=> true

# Special values are objects
nil # equivalent to null in other languages
true # truth
false # falsehood

nil.class #=> NilClass
true.class #=> TrueClass
false.class #=> FalseClass

# Equality
1 == 1 #=> true
2 == 1 #=> false

# Inequality
1 != 1 #=> false
2 != 1 #=> true

# apart from false itself, nil is the only other 'falsey' value

!!nil   #=> false
!!false #=> false
!!0     #=> true
!!""    #=> true

# More comparisons
1 < 10 #=> true
1 > 10 #=> false
2 <= 2 #=> true
2 >= 2 #=> true

# Combined comparison operator (returns `1` when the first argument is greater, 
# `-1` when the second argument is greater, and `0` otherwise)
1 <=> 10 #=> -1
10 <=> 1 #=> 1
1 <=> 1 #=> 0

1 == 1.0 #=> true
1.eql?(1.0) #=> false 如果接收器和参数具有相同的类型和相等的值，则返回 true

aObj == bObj #=> true
a.equal?(bObj) #=> false
a.equal?(aObj) #=> true

# Logical operators
true && false #=> false
true || false #=> true
!true #=> false

# There are alternate versions of the logical operators with much lower
# precedence. These are meant to be used as flow-control constructs to chain
# statements together until one of them returns true or false.

# `do_something_else` only called if `do_something` succeeds.
do_something() and do_something_else()
# `log_error` only called if `do_something` fails.
do_something() or log_error()

# String interpolation

placeholder = 'use string interpolation'
"I can #{placeholder} when using double quoted strings"
#=> "I can use string interpolation when using double quoted strings"

# You can combine strings using `+`, but not with other types
'hello ' + 'world'  #=> "hello world"
'hello ' + 3 #=> TypeError: can't convert Fixnum into String
'hello ' + 3.to_s #=> "hello 3"
"hello #{3}" #=> "hello 3"

# Combine strings and operators
'hello ' * 3 #=> "hello hello hello "

# Append to string
'hello' << ' world' #=> "hello world"

# print to the output with a newline at the end
puts "I'm printing!"
#=> I'm printing!
#=> nil

# print to the output without a newline
print "I'm printing!"
#=> I'm printing! => nil

# here document
print <<EOF
    This is how you create a mulitple
    line string. Anything inbetween is
    part of the string.
EOF

# Variables
x = 25 #=> 25
x #=> 25

# Note that assignment returns the value assigned
# This means you can do multiple assignment:

x = y = 10 #=> 10
x #=> 10
y #=> 10

a, b, c = 10, 20, 30

a, b = b, c

# By convention, use snake_case for variable names
snake_case = true

# Use descriptive variable names
path_to_project_root = '/good/name/'
m = '/bad/name/'

# Symbols are immutable, reusable constants represented internally by an
# integer value. They're often used instead of strings to efficiently convey
# specific, meaningful values

:pending.class #=> Symbol

status = :pending

status == :pending #=> true

status == 'pending' #=> false

status == :approved #=> false

# Strings can be converted into symbols and vice versa:

status.to_s #=> "pending"
"argon".to_sym #=> :argon

# Arrays

# This is an array
array = [1, 2, 3, 4, 5] #=> [1, 2, 3, 4, 5]

# Arrays can contain different types of items

[1, 'hello', false] #=> [1, "hello", false]

# Arrays can be indexed
# From the front
array[0] #=> 1
array.first #=> 1
array[12] #=> nil

# Like arithmetic, [var] access
# is just syntactic sugar
# for calling a method [] on an object
array.[] 0 #=> 1
array.[] 12 #=> nil

# From the end
array[-1] #=> 5
array.last #=> 5

# With a start index and length
array[2, 3] #=> [3, 4, 5]

# Reverse an Array
a = [1,2,3]
a.reverse! #=> [3,2,1]

# Or with a range
array[1..3] #=> [2, 3, 4]

# Add to an array like this
array << 6 #=> [1, 2, 3, 4, 5, 6]
# Or like this
array.push(6) #=> [1, 2, 3, 4, 5, 6]

# Check if an item exists in an array
array.include?(1) #=> true

# Hashes are Ruby's primary dictionary with key/value pairs.
# Hashes are denoted with curly braces:
hash = { 'color' => 'green', 'number' => 5 }

hash.keys #=> ['color', 'number']

# Hashes can be quickly looked up by key:
hash['color'] #=> 'green'
hash['number'] #=> 5

# Asking a hash for a key that doesn't exist returns nil:
hash['nothing here'] #=> nil

# When using symbols for keys in a hash, you can use this alternate syntax:

new_hash = { defcon: 3, action: true }

new_hash.keys #=> [:defcon, :action]

# Check existence of keys and values in hash
new_hash.key?(:defcon) #=> true
new_hash.value?(3) #=> true

# Tip: Both Arrays and Hashes are Enumerable
# They share a lot of useful methods such as each, map, count, and more

# Control structures

if true
  'if statement'
elsif false
  'else if, optional'
else
  'else, also optional'
end


# In Ruby, traditional `for` loops aren't very common. Instead, these 
# basic loops are implemented using enumerable, which hinges on `each`:

(1..5).each do |counter|
  puts "iteration #{counter}"
end

# Which is roughly equivalent to this, which is unusual to see in Ruby:

for counter in 1..5
  puts "iteration #{counter}"
end

# The `do |variable| ... end` construct above is called a “block”. Blocks are similar
# to lambdas, anonymous functions or closures in other programming languages. They can
# be passed around as objects, called, or attached as methods. 
#
# The "each" method of a range runs the block once for each element of the range.
# The block is passed a counter as a parameter.

# You can also surround blocks in curly brackets:
(1..5).each { |counter| puts "iteration #{counter}" }

# The contents of data structures can also be iterated using each.
array.each do |element|
  puts "#{element} is part of the array"
end

hash.each do |key, value|
  puts "#{key} is #{value}"
end

# If you still need an index you can use "each_with_index" and define an index
# variable
array.each_with_index do |element, index|
  puts "#{element} is number #{index} in the array"
end

counter = 1
while counter <= 5 do
  puts "iteration #{counter}"
  counter += 1
end
#=> iteration 1
#=> iteration 2
#=> iteration 3
#=> iteration 4
#=> iteration 5

# There are a bunch of other helpful looping functions in Ruby,
# for example "map", "reduce", "inject", the list goes on. Map,
# for instance, takes the array it's looping over, does something
# to it as defined in your block, and returns an entirely new array.
array = [1,2,3,4,5]
doubled = array.map do |element|
  element * 2
end
puts doubled
#=> [2,4,6,8,10]
puts array
#=> [1,2,3,4,5]

grade = 'B'

case grade
when 'A'
  puts 'Way to go kiddo'
when 'B'
  puts 'Better luck next time'
when 'C'
  puts 'You can do better'
when 'D'
  puts 'Scraping through'
when 'F'
  puts 'You failed!'
else
  puts 'Alternative grading system, eh?'
end
#=> "Better luck next time"

# cases can also use ranges
grade = 82
case grade
when 90..100
  puts 'Hooray!'
when 80...90
  puts 'OK job'
else
  puts 'You failed!'
end
#=> "OK job"

# exception handling:
begin
  # code here that might raise an exception
  raise NoMemoryError, 'You ran out of memory.'
rescue NoMemoryError => exception_variable
  puts 'NoMemoryError was raised', exception_variable
rescue RuntimeError => other_exception_variable
  puts 'RuntimeError was raised now'
else
  puts 'This runs if no exceptions were thrown at all'
ensure
  puts 'This code always runs no matter what'
end

# Methods

def double(x)
  x * 2
end

# Methods (and blocks) implicitly return the value of the last statement
double(2) #=> 4

# Parentheses are optional where the interpretation is unambiguous
double 3 #=> 6

double double 3 #=> 12

def sum(x, y)
  x + y
end

# Method arguments are separated by a comma
sum 3, 4 #=> 7

sum sum(3, 4), 5 #=> 12

# yield
# All methods have an implicit, optional block parameter
# it can be called with the 'yield' keyword

def surround
  puts '{'
  yield
  puts '}'
end

surround { puts 'hello world' }

# {
# hello world
# }


# Blocks can be converted into a `proc` object, which wraps the block 
# and allows it to be passed to another method, bound to a different scope,
# or manipulated otherwise. This is most common in method parameter lists,
# where you frequently see a trailing `&block` parameter that will accept 
# the block, if one is given, and convert it to a `Proc`. The naming here is
# convention; it would work just as well with `&pineapple`:
def guests(&block)
  block.class #=> Proc
  block.call(4)
end

# The `call` method on the Proc is similar to calling `yield` when a block is 
# present. The arguments passed to `call` will be forwarded to the block as arugments:

guests { |n| "You have #{n} guests." }
# => "You have 4 guests."

# You can pass a list of arguments, which will be converted into an array
# That's what splat operator ("*") is for
def guests(*array)
  array.each { |guest| puts guest }
end

# Destructuring

# Ruby will automatically destrucure arrays on assignment to multiple variables:
a, b, c = [1, 2, 3]
a #=> 1
b #=> 2
c #=> 3

# In some cases, you will want to use the splat operator: `*` to prompt destructuring
# of an array into a list:

ranked_competitors = ["John", "Sally", "Dingus", "Moe", "Marcy"]

def best(first, second, third)
  puts "Winners are #{first}, #{second}, and #{third}."
end

best *ranked_competitors.first(3) #=> Winners are John, Sally, and Dingus.

# The splat operator can also be used in parameters:
def best(first, second, third, *others)
  puts "Winners are #{first}, #{second}, and #{third}."
  puts "There were #{others.count} other participants."
end

best *ranked_competitors 
#=> Winners are John, Sally, and Dingus.
#=> There were 2 other participants.

# By convention, all methods that return booleans end with a question mark
5.even? # false
5.odd? # true

# And if a method ends with an exclamation mark, it does something destructive
# like mutate the receiver. Many methods have a ! version to make a change, and
# a non-! version to just return a new changed version
company_name = "Dunder Mifflin"
company_name.upcase #=> "DUNDER MIFFLIN"
company_name #=> "Dunder Mifflin"
company_name.upcase! # we're mutating company_name this time!
company_name #=> "DUNDER MIFFLIN"


# Define a class with the class keyword
class Human

  # A class variable. It is shared by all instances of this class.
  @@species = 'H. sapiens'

  # Basic initializer
  def initialize(name, age = 0)
    # Assign the argument to the "name" instance variable for the instance
    @name = name
    # If no age given, we will fall back to the default in the arguments list.
    @age = age
  end

  # Basic setter method
  def name=(name)
    @name = name
  end

  # Basic getter method
  def name
    @name
  end

  # The above functionality can be encapsulated using the attr_accessor method as follows
  attr_accessor :name

  # Getter/setter methods can also be created individually like this
  attr_reader :name
  attr_writer :name

  # A class method uses self to distinguish from instance methods.
  # It can only be called on the class, not an instance.
  def self.say(msg)
    puts msg
  end

  def species
    @@species
  end
end


# Instantiate a class
jim = Human.new('Jim Halpert')

dwight = Human.new('Dwight K. Schrute')

# Let's call a couple of methods
jim.species #=> "H. sapiens"
jim.name #=> "Jim Halpert"
jim.name = "Jim Halpert II" #=> "Jim Halpert II"
jim.name #=> "Jim Halpert II"
dwight.species #=> "H. sapiens"
dwight.name #=> "Dwight K. Schrute"

# Call the class method
Human.say('Hi') #=> "Hi"

# Variable's scopes are defined by the way we name them.
# Variables that start with $ have global scope
$var = "I'm a global var"
defined? $var #=> "global-variable"

# Variables that start with @ have instance scope
@var = "I'm an instance var"
defined? @var #=> "instance-variable"

# Variables that start with @@ have class scope
@@var = "I'm a class var"
defined? @@var #=> "class variable"

# Variables that start with a capital letter are constants
Var = "I'm a constant"
defined? Var #=> "constant"

# Class is also an object in ruby. So class can have instance variables.
# Class variable is shared among the class and all of its descendants.

# base class
class Human
  @@foo = 0

  def self.foo
    @@foo
  end

  def self.foo=(value)
    @@foo = value
  end
end

# derived class
class Worker < Human
end

Human.foo # 0
Worker.foo # 0

Human.foo = 2 # 2
Worker.foo # 2

# Class instance variable is not shared by the class's descendants.

class Human
  @bar = 0

  def self.bar
    @bar
  end

  def self.bar=(value)
    @bar = value
  end
end

class Doctor < Human
end

Human.bar # 0
Doctor.bar # nil

module ModuleExample
  def foo
    'foo'
  end
end

# Including modules binds their methods to the class instances
# Extending modules binds their methods to the class itself

class Person
  include ModuleExample
end

class Book
  extend ModuleExample
end

Person.foo     # => NoMethodError: undefined method `foo' for Person:Class
Person.new.foo # => 'foo'
Book.foo       # => 'foo'
Book.new.foo   # => NoMethodError: undefined method `foo'

# Callbacks are executed when including and extending a module

module ConcernExample
  def self.included(base)
    base.extend(ClassMethods)
    base.send(:include, InstanceMethods)
  end

  module ClassMethods
    def bar
      'bar'
    end
  end

  module InstanceMethods
    def qux
      'qux'
    end
  end
end

class Something
  include ConcernExample
end

Something.bar     # => 'bar'
Something.qux     # => NoMethodError: undefined method `qux'
Something.new.bar # => NoMethodError: undefined method `bar'
Something.new.qux # => 'qux'
```

# References
1. [Learn X in Y minutes](https://learnxinyminutes.com/docs/ruby/)
