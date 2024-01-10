# Object Oriented Programming
Based on the concept of objects, which are data structures that contain data in the form of fields, and code in teh form of procedures.
Uses objects and classes.

# Classes and Objects
A class is a blueprint for creating objects - it defines the properties and behaviors of an object. An object is an instance of a class - a concrete entity based on a class.

Example: a Book class that defines what a book is and what can be done to a book.

Defining classes in Ruby (the name of the class should be PascalCase):

class Book
end

To create an instance or object of the Book class (the new method is a method that is defined in the Class class; it creates a new instance of the class it is called on):

book = Book.new

Including properties and behaviors:

class Book
  def initialize(title, author, pages)
    @title = title
    @author = author
    @pages = pages
  end

  def print_title
    puts @title
  end

  def print_author
    puts @author
  end

  def print_pages
    puts @pages
  end
end

To define a method we use the def keyword. The name of the method should be in snake_case. These are also called instance methods.

The @ symbol is used to define instance variables.

The initialize method is a special method called when an instance of a class is created. It initializes the instance variables of the class. It's also called the constructor method.
We can pass arguments to the initialize method when we create an instance of a class. 

book = Book.new("The Pragmatic Programmer", "Andy Hunt and Dave Thomas", 352)

book.print_title # => The Pragmatic Programmer

book.print_author # => Andy Hunt and Dave Thomas

book.print_pages # => 352


# Inheritance
An inheritance is a mechanism that allows us to define a class based on another class - we can create a class that is a specialization of another class.
The class inherited from is called the superclass or parent class.
The class the inherits is called the subclass or child class.

Example: a Book class that inherits from an Item class

class Item
  def initialize(name, price)
    @name = name
    @price = price
  end

  def print_name
    puts @name
  end

  def print_price
    puts @price
  end
end

class Book < Item
  def initialize(name, price, author, pages)
    super(name, price)
    @author = author
    @pages = pages
  end

  def print_author
    puts @author
  end

  def print_pages
    puts @pages
  end
end

The Book class inherits all the properties and behaviors of the Item class. 
The super keyword is used to call a method on the parent class with the same name as the method that calls super. (In the class Book, the initialize method uses super to call the initialize method from the Item class.)
If super is used without arguments, it will pass along the arguments used in the child method call to the parent method.
If super is used with just parenthesis - () - this says to call the method without passing arguments from the child method to the parent method.

Creating an instance of the Book class and calling the methods:
book = Book.new("The Pragmatic Programmer", 29.95, "Andy Hunt and Dave Thomas", 352)

book.print_name # => The Pragmatic Programmer

book.print_price # => 29.95

book.print_author # => Andy Hunt and Dave Thomas

book.print_pages # => 352


# Polymorphism
The ability of an object to take on many forms. The object can respond to the same method in different ways. 
Example:
class Animal
  def speak
    puts "I am an animal"
  end
end

class Dog < Animal
  def speak
    puts "I am a dog"
  end
end

class Cat < Animal
  def speak
    puts "I am a cat"
  end
end

def speak(animal)
  animal.speak
end

animal = Animal.new

speak(animal) # => I am an animal

dog = Dog.new

speak(dog) # => I am a dog

cat = Cat.new

speak(cat) # => I am a cat

Example from Code Coach Joe:
class Toy
  def play
    raise NotImplementedError, 'Each toy must implement the play method'
  end
end

class Car < Toy
  def play
    'Vroom, vroom!'
  end
end

class Doll < Toy
  def play
    'Let\'s have a tea party.'
  end
end

class ActionFigure < Toy
  def play
    'To the rescue!'
  end
end

toys = [Car.new, Doll.new, ActionFigure.new]
toys.each do |toy|
  puts toy.play
end

In this code, each type of toy is a subclass of Toy and implements the play method. When we iterate over the array of toys and call play on each one, the correct method is called for each type of toy. This is polymorphism.


# Encapsulation and Access Modifiers
Encapsulation is the process of hiding the internal details of an object. Then the internal details of an object can only be accessed through a well-defined interface.

The instance variables inside of a class are private and can only be accessed by instance methods in the class.

class Book
  def initialize(title)
    @title = title
  end

  def print_title
    puts @title
  end
end

book = Book.new("Book Title")

book.print_title # => Book Title

Access modifiers are keywords that are used to control the visibility of methods and variables: public, protected, private
Public: can be accessed by anyone
-by default, methods are public unless we use the private or protected keyword
-describe the outside behaviors of an object
-are called with the object as the explicit receiver (the thing we're calling the method on)

class Person 
  def speak
    puts "Hey, Tj!"
  end 
end 
you = Person.new 
you.speak # "Hey, Tj!"

Private: can only be accessed by the class that defines them
-for internal usage
-the only way to have external access is to call it within a public method
-cannot be called with an explicit receiver, the reciever is always implicitly self
-can think of as internal helper methods

class Person
  def speak
    puts "Hey, Tj!"
  end
  def whisper_louder
    whisper
  end 
comment # private methods are for internal usage within the defining class
  private 
  def whisper
    puts "His name's not really 'Tj'." 
  end 
end
you = Person.new 
you.speak # "Hey, Tj!"
a_hater = Person.new
a_hater.speak # "Hey, Tj!"
a_hater.whisper # NoMethodError
a_hater.whisper_louder # "His name's not really 'Tj'."

Protected: can only be accessed by the class that defines them and its subclasses
-similar to private with the addition that it can be called with or without an explicit reveiver but that receiver is always self (its defining class) or an object that inherits from self


# Getters and Setters
Getters and setters are methods that are used to get and set the value of instance variables. 
class Book
  def initialize(title, author, pages)
    @title = title
    @author = author
    @pages = pages
  end

  def title
    @title
  end

  def title=(title)
    @title = title
  end

  def author
    @author
  end

  def author=(author)
    @author = author
  end

  def pages
    @pages
  end

  def pages=(pages)
    @pages = pages
  end
end

book = Book.new("The Pragmatic Programmer", "Andy Hunt and Dave Thomas", 352)

puts book.title # => The Pragmatic Programmer

puts book.author # => Andy Hunt and Dave Thomas

puts book.pages # => 352

book.title = "The Pragmatic Programmer 2"

puts book.title # => The Pragmatic Programmer 2


# Accessing modifiers with attr_accessor
These are like a shortcut to writing out the getter and setter methods in the class.

The attr_accessor method is used to define both reader and writer methods
The attr_reader creates only the reader.
The attr_writer creates only the writer.

class Book
  attr_accessor :title, :author, :pages

  def initialize(title, author, pages)
    @title = title
    @author = author
    @pages = pages
  end
end

book = Book.new("The Pragmatic Programmer", "Andy Hunt and Dave Thomas", 352)

puts book.title # => The Pragmatic Programmer

puts book.author # => Andy Hunt and Dave Thomas

puts book.pages # => 352

book.title = "The Pragmatic Programmer 2"

puts book.title # => The Pragmatic Programmer 2


# Example RSpec test for the Book class
require 'rspec'

require_relative '../book'

describe Book do
  describe '#title' do
    it 'returns the title of the book' do
      book = Book.new("The Pragmatic Programmer", "Andy Hunt and Dave Thomas", 352)

      expect(book.title).to eq("The Pragmatic Programmer")
    end
  end

  describe '#author' do
    it 'returns the author of the book' do
      book = Book.new("The Pragmatic Programmer", "Andy Hunt and Dave Thomas", 352)

      expect(book.author).to eq("Andy Hunt and Dave Thomas")
    end
  end

  describe '#pages' do
    it 'returns the number of pages in the book' do
      book = Book.new("The Pragmatic Programmer", "Andy Hunt and Dave Thomas", 352)

      expect(book.pages).to eq(352)
    end
  end
end


# Modules and Mixins
Modules are used to group related methods, classes, and constants together. They are great for organizing code and for sharing code between classes.

Example of a module:
module Math
  def self.square(x)
    x * x
  end
end

puts Math.square(2) # => 4

We can include or extend a module in a class.

If we include the module:
class Book
    include Math
...

Then we can use the square method from the module like this:
puts book.square(2) # => 4

If we extend the module:
class Book
    extend Math
...

Then we can use the square method on the Book class:
puts Book.square(2) # => 4


# Class methods and class variables
Class methods are methods that are defined in a class and can be called on the class itself. 
Class variables are variables that are defined in a class and can be accessed by all the class methods in the class. We use the @@ symbol to define class variables.

Examples:
class Book
  @@count = 0

  def initialize(title, author, pages)
    @title = title
    @author = author
    @pages = pages

    @@count += 1
  end

  def self.count
    @@count
  end
end

book1 = Book.new("The Pragmatic Programmer", "Andy Hunt and Dave Thomas", 352)

book2 = Book.new("The Pragmatic Programmer 2", "Andy Hunt and Dave Thomas", 352)

puts Book.count # => 2


The self keyword in class methods refers to the class itself.

example:
class User

  def self.say_hello
    puts "Hello"
  end
end

User.say_hello # => Hello

The self keyword in instance methods refers to teh instance of the class that the method is called on.
example:
class User
  def initialize(name)
    @name = name
  end

  def say_hello
    puts "Hello, my name is #{self.name}"
  end

  def name
    @name
  end
end

user = User.new("Alice")

user.say_hello # => Hello, my name is Alice


# Video: Basic Ruby - Debugging
Errors will tell us the lines of the errors, an error message, and will sometimes give suggestions.


# Video: Basic Ruby - Classes
A class is essentially a template.

class Party
  @@attendee_count = 0
  @@attendee_list = []

  def add_attendee(name)
    @@attendee_count += 1
    @@attendee_list.push(name)
  end

  def print_attendees
    puts "Total Attendee Count: #{@@attendee_count}"
    puts "Attendee Names:"
    @@attendee_list.each {|attendee| puts attendee}
  end
end

party_1 = Party.new()
party_1.add_attendee("Luci")
part_1.print_attendees

party_2 = Party.new
party_2.print_attendees # => This will output the same list of attendees as party_1 because we're using class variables. 
A class variable is a variable on every instance of a class.
If we want the variables to be unique to each instance of the class, we instead need to use instance variables. An instance variable will be specific to the one instance of a class that it's attached to. (We just use one @ symbol for instance variables.)

We use initialize which is similar to a constructor.

class Party
  def initialize
    @attendee_count = 0
    @attendee_list = []
  end

  def add_attendee(name)
    @attendee_count += 1
    @attendee_list.push(name)
  end

  def print_attendees
    puts "Total Attendee Count: #{@attendee_count}"
    puts "Attendee Names:"
    @attendee_list.each {|attendee| puts attendee}
  end
end

Then printing the list for party_1 and party_2 should result in different lists. (Once we've added names to the list array.)

We can use the attr_accessor to access the values of the varibles we create.

class Party
  attr_accessor "attendee_count, :attendee_list

...

puts party_1.attendee_count # => the count number