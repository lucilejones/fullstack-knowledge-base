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
If super is used without arguments, it will pass along the arguments used for the original method call to the new one. 
If super is used with just parenthesis - () - this says to call the method without passing arguments from the child method to the parent method.

Creating an instance of the Book class and calling the methods:
book = Book.new("The Pragmatic Programmer", 29.95, "Andy Hunt and Dave Thomas", 352)

book.print_name # => The Pragmatic Programmer

book.print_price # => 29.95

book.print_author # => Andy Hunt and Dave Thomas

book.print_pages # => 352

