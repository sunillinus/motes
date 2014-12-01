#Ruby Bits

##Conditional Assignment

name ||= ‘rhino’

Conditional Return values

if name
	x = name + ‘ woof’
else
	x = ‘rhino’
end

to

x = if name
	name + ‘ woof’
else
	‘rhino’
end

Conditional statements return a value (in Ruby everything returns a value).

Exceptions

raise AuthorizationException.new

begin
rescue AuthorizationException
end


Splat Arguments
mention(status, *names)  => names is an array

compact removes nil from arrays

Classes

private method - only accessible from inside instance.
protected methods - also from other instances of same class

super calls the corresponding method in the class hierarchy. Will pass args automatically.

Blah.ancestors to get list of superclasses (and mixins). In ruby all objects inherit from Object which inherit from BasicObject.

Method overriding is faster than case/when, so set the typical behavior in the  parent class and override when required instead of case/when

Ruby Classes
Ruby classes are really objects of class Class. When you define a class (class MyClass), ruby creates an instance of class Class and global variable/constant with the name MyClass that points to that instance. When you call MyClass.new, you are calling new method on the ‘MyClass’ instance of class Class. Which returns an instance of class MyClass


Singleton class (anonymous class/meta class)
Not to be confused with singleton pattern.
In Ruby objects can have methods independent of parent class definition. They are called singleton methods and they are held in a metaclass called singleton class
If you add a method to a specific object (def foobar.size; 1; end) - a singleton method, Ruby inserts a new anonymous class into the inheritance hierarchy as a container to hold these methods. http://www.devalot.com/articles/2008/09/ruby-singleton
anObject.extend(SomeModule), will add the methods in the module to the object’s singleton class.
Adding singleton methods directly:
	class << foobar
		def foo
			“hello”
		end
	end
class << foobar refers to the singleton class of the foobar object. so
class << self refers to current self’s singleton class (so in effect self is redirected to point to the singleton class.
class << self; self; end returns the singleton class of whatever the outer self is. So -
class Object
	def metaclass
		class << self
			self
		end
	end
end

is one way to access an object’s metaclass (from ruby1.9 you can use singleton_class method).

Singleton methods is used for Class Methods
Ruby classes are really instances of the class Class. And the class name is just a constant that points to the instance (of class Class).
Ruby doesn’t have class methods, instead class methods are really singleton methods (added to the singleton class of the instance of class Class).

Singleton class TLDR -
Every Ruby object has a metaclass associated with it where any methods defined specifically for that instance is stored. These methods are called singleton methods and the metaclass the singleton class. So if I do -
def foo.goo
	“goo”
end
the goo method is stored in the singleton class for foo. You can access the singleton class with the ’  singleton_class' method and the 'singleton methods' with singleton_methods method.

Metaclass/singleton class can’t be instantiated.

Class methods in Ruby
Ruby does’t really have class or static/methods. Instead class methods are singleton methods of the instance of the class object for that class (in Ruby a class is an object of class Class - class name is simply a constant that points to instance of class Class). So if I do -
class Foo
	def self.goo
		“goo”
	end
end
goo is a singleton method stored in the singleton class of the Goo class object (instance of class Class).
class << self refers to the singleton class of self (whatever self is). The class << self syntax changes the self to point to the singleton class of the current object (self).

class_eval
class_eval on a class allows us to add an instance method to that class (dynamically without reopening the class).
class_eval is a method of the Module class (and the receiver has to be a module or a class - and not any object).

instance_eval
instance_eval runs code as if from inside a specific instance (has access to instance variables and private methods).
Now if you def a method using instance_eval, it is added to that specific instance - or in reality to the singleton class.
instance_eval adds a method to the singleton class of the object it is called on -
p = Person.new
p.instance_eval do
	def name() “foo” end
end
p.name => “foo”

Now if instance_eval is invoked on a class (Person.instance_eval) to add a method, the method is added to the instance of class Class (that the Person points to). In effect it is added to the singleton class of that instance of class Class and in turn you are creating a class/static method. This is little counter intuitive, cause when called on a class, class_eval will add an instance method and instance_eval with add a class method.
http://www.jimmycuadra.com/posts/metaprogramming-ruby-class-eval-and-instance-eval
http://www.stanford.edu/~ouster/cgi-bin/cs142-winter14/classEval.php

ActiveSupport

gem install activesupport
gem install i18n <———————not a depeendency, but helpfull

require ‘active_support/all’

Core extensions Arrays - from(x), to(x), in_groups_of(x), split(x)

Core extensions Hash - diff({}), strigify_keys, reverse_merge({}), except, assert_valid_keys

Integer - odd?, even?
1.ordinalize -> “1st"
pluralize, sigularize, titleize, humanize


Modules

can club related functions in a file and require that file. But the method names will be in the global namespace - NOT GOOD
Instead you wrap them in a module (ex: module ImageUtils) and call the methods via the module (ImageUtils.blah). The methods have to be “class methods” (def self.blah).
You can wrap classes, constants and even other modules to namespace them.
You use constant lookup operator :: to restrict scope to a module. eg: A::B

Mixins

Include a module in a  class, the module methods becomes available in the class as instance methods and the method has access to the class instance variables.
Mixin is part of the class’s ancestor chain (ClassBlah.ancestors, use ClassBlah.included_modules to list just mixins). Kernel is a mixed into Object (which inherits from BasicObject

Inheritance vs mixins
Ruby doesn’t allow multiple inheritance, so you use mixins to achieve it.
Inheritance implies specialization (class Image < Media, but not really Image < Sharable).
Use mixin to add shared functionality (class Image; include Sharable; end)

Extend
Use extend to add a module as class methods (vs include which adds them as instance methods).
You also call extend from an instance to add the module methods as instance methods. ex - image = Image.new; image.extend(Sharable). These methods are added to a singleton class.
Object#extend does what Module#include does but on the object’s singleton class.
extend does an instance_eval - while called from a class it adds class methods (singleton methods) and when called from an object, it adds per specific methods (singleton methods again)
while include does class_eval - adds instance methods when called from a class)

Method Hooks
self.included is called when module is included.
Regular pattern in Ruby (gems) to use this hook to add instance and class methods to a class that includes the module -
module MyMod
  def self.included(target)
    target.send(:include, InstanceMethods)
    target.extend ClassMethods
    target.class_eval do
      a_class_method
    end
  end

  module InstanceMethods
    def an_instance_method
    end
  end

  module ClassMethods
    def a_class_method
      puts "a_class_method called"
    end
  end
end

class MyClass
include MyMod;
end

ActiveSupport:: Concern encapsulates this pattern  and provides an include block (it comes from concern in Aspect Oriented Programming)-
module TagLib
  extend ActiveSupport::Concern

  module ClassMethods
    def find_by_tags()
      # ...
    end
  end

  module InstanceMethods
    def tags()
      # ...
    end
  end
end

class ActiveRecord::Base
  include TagLib
end

Ruby Blocks

You can pass a block of code to a method in ruby and the code will be executed wherever there is a yield:

def call_a_block
	yield
	yield
end

call_a_block { puts “foo”}   # can use do end too.

Can pass a variable to the executed block:

def call_a_block
	yield “foo”
end

call_a_block {|x| puts x}

Yield returns the value returned by the block.

Procs and Lambdas

proc or lambda stores  block of code.

my_proc = Proc.new { puts “foo”} or my_proc = lambda { puts “boo” }.
From 1.9: my_proc = -> {puts “foo”} # staby lambda

To invoke a proc/lambda: my_proc.call

When invoking a method with a proc, you have to use &proc to convert it to a block (so yield works).
If you also want to assign proc to a variable in the called method, define the method with &param

proc = -> { |tweet| puts tweet }
tweet.each &proc
def each(&block)
	puts block # <- proc
	yield
	# &block to convert it back to a block

end

tweets.map {|tweet| tweet.user} <—> tweets.map(&:user) but not this tweets.map(&:user.name)

A method can use block_given? to detect if a block is passed to a method

To init an object with a  block:
class Tweet
	def initialize
		yield self if block_given?
	end
end

Current state of vars is preserved when a lambda/proc is created, so you can use it form closure

Struct

Same as class, but better for classes with just data  -
Person = Struct.new(:name, :gender, :dob)

To add a method using struct, pass a block -

Person = Struct.new(:name, :gender, :dob) do
	def to_s
	end
end

alias_method :alias, :method

define_method - to define methods dynamically

class Game
  SYSTEMS = ['SNES', 'PS1', 'Genesis']
  attr_accessor :name, :year, :system
  SYSTEMS.each do |system|
    define_method "runs_on_#{system.downcase}?" do
      self.system == system
    end
  end
end

To pass args to the defined method, specify it as args to the block -

define_method(:foo) {|x| puts x }

send - to call methods dynamically

tweet.send(:post) or tweet.send(‘post’) # send can invoke both private and protected methods. public_send can’t.

to pass args to send method:

blah.send(:method_name, arg1, arg2) or using splat - blah.send(:method_name, *[arg1, arg2])

Passing blocks using define and send
class Library
  attr_accessor :games
  [:each, :map, :select].each do |f|
    define_method f do |&block|
      games.send f, &block
    end
  end
end

method method - to store a method (methods are also objects in ruby) as a proc.
name_method = person.method(:name)
name_method.call

Self
def Tweet.find is same as def self.find, bit not common

when a method is called with no explicit receivers, the method is executed on the object self points to.

Inside a class_eval, self points to the class it is called on
