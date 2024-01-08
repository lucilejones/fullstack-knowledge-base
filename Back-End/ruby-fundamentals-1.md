# Online code editor and IDE
Replit
https://replit.com/

(We will eventually install Ruby on our computers.)

Create a new Replit by clicking on "Create repl"
Choose Ruby as the template
Click the "Run" button to run code

Ruby files have the extension .rb
(main.rb, app.rb)

to run the file (in the terminal, or in the shell in Replit):
ruby file_name
ex. ruby main.rb

for future: user input will always be strings, so we need to convert user input to integers, floats, etc

Ruby uses snake_case


# Comments
In Ruby, comments are created with the # symbol

# Data Types
A data type is a classification that dictates what sort of value a variable can hold and how the computer interprets that value.

# Strings
In Ruby, single-quote strings are simpler and more literal; best used when we need a string exactly as it is.
Example: puts 'Hello\nWorld' will output Hello\nWorld
It doesn't interpret \n as a newline character.
Double-quote strings are more flexible and allow for escape sequences and string interpolation.
Escape sequences are special characters that are preceded by a backslash (\). 
\n represents a newline character, \t a tab, \\ a backslash.
Example: 
name = "Ruby"
puts "Hello, #{name}!\nWelcome to programming!"
# Output: Hello, Ruby!
#         Welcome to programming!

# Numbers
In Ruby, numbers fall into two categories: integers and floating-point numbers

# Booleans
true: a Ruby object that represents truth; it is an instance of the TrueClass
false: represents falsehood and is an instance of the FalseClass
In Ruby only false and nil (nothing, no value) are treated as false in conditional expressions. Every other value, including 0, "", and [] is true.

# Arrays
Arrays can be created with square brackets:
numbers = [1, 2, 3, 4]
or using the Array.new method (this method creates an array with specified size and default value):
empty_array = Array.new(3) # => [nil, nil, nil]
zeros = Array.new(3, 0) # => [0, 0, 0]

Arrays are zero-indexed and elements are accessed with their index:
array = [10, 20, 30, 40, 50]
puts array[0] # => 10
puts array[-1] # => 50 (last element)

Arrays can be modifiied by adding elements, removing, replacing, etc.

# Hashes
Also known as associative arrays or dictionaries in other languages are collections of key-value pairs. They are similar to objects (they store data in a structured way), but they are not the same as objects created from custom classes.
Each entry consists of a key and a value. The key is used to retrieve the corresponding value.
Hash keys and values can be objects of any type: numbers, strings, arrays, etc.
my_hash = {
    'name' => 'Alice', 
    'age' => 30,
    'is_student' => true
}
We access or update hash values using their keys:
puts my_hash['name'] # => Alice
my_hash['age'] = 31 # updated the age

Ruby allows the use of symbols as hash keys:
student = {
    :name => 'Bob',
    :grade => 'A'
}
Newer syntax:
student = {
    name: 'Bob',
    grade: 'A'
}

# 'nil' in Ruby
nil is a value that represents the concept of "nothing" or "no value"
Common usage: it's commonly returned by methods that have no meaningful value to return, or when an operation (such as searching an array) does not yield a result.

# Variables
Ruby is dynamically typed, meaning we don't need to declare a variable's type beforehand
variables are declared with snake_case - lowercase letters with underscores

Local Variables: scope is limited to the block, method or class in which they're defined.
-begin with a lowercase or underscore
local_var = "I'm local"

Constant Variables: value cannot be changed once assigned
-begin with an uppercase letter
CONSTANT_VAR = 3.14

# Arithmetic Operators
addition: +
subtraction: -
multiplication: *
division: /
(In Ruby, dividing two integers always results in an integer - it will truncate any decimal part; in order to get a decimal result, at least one of the operands must be a float)
exponentiation: **
modulus: %

Variables can be used in place of literal numbers:
x = 10
y = 5
sum = x + y # => 15

Compound assignment operators:
a += 5 # equivalent to a = a + 5
a -= 2 # equivalent to a = a - 2

Strings can be concatenated with + or <<
Difference: + creates a new string; << modifies the original string
first_name = "Jane"
last_name = "Doe"
full_name = first_name + " " + last_name # => "Jane Doe"
first_name << last_name # => "JaneDoe"

We don't typically use the shovel operator << with strings.

String interpolation (allows us to embed Ruby expressions within a string):
full_name = "#{first_name} #{last_name}" # => "Jane Doe"

In Ruby, trying to add a number directly to a string will result in a TypeError because Ruby doesn't implicitly convert types during arithmetic.
"Hello" + 3 # => TypeError

# Conditional Operators and Control Flow
==, !=, >, <, >=, <=, <=>
combined comparison <=> : Checks if the first value is less than, equal to, or greater than the second. It returns 0 if the values are equal, 1 if the first value is greater, and -1 if the first value is less.

# Logical Operators
and &&
or ||
not !

ternary operator: condition ? true_value : false_value
The '?' represents the if, and the ':' represents the else. The condition is evaluated, and if it is true, the true_value is returned; otherwise the false_value is returned.

# Conditional Statements
In Ruby, these are created using if, if/else, if/elsif/else, and case statements

if statement: the most basic; runs the code block only if the condition is true

if condition
  # code to exectue if condition is true
end

temperature = 30
if temperature > 25
    puts "It's a hot day!"
end

if/else statement: can execute different code blocks based on a condition; if the condition is true, the if block is executed, if it's false, the else block is executed.

if temperature > 25
    puts "It's a hot day!"
else
    puts "It's not a hot day."
end

elsif statement: allows us to check multiple conditions; used in conjuctions with the if statement and must come before the else block

if condition1
    # code to execute if condition1 is true
elsif condition2
    # code to execute if condition2 is true
else
    # code to execute if both conditions are false
end

temperature = 30
if temperature > 25
  puts "It's a hot day!"
elsif temperature < 10
  puts "It's a cold day!"
else
  puts "It's not a hot day."
end

unless statement: the opposite of the if statement; executes the code block only if the condition is false

age = 18
uless age < 18
    puts "You can vote!"
end

case statement: used to compare a value against multiple conditons; similar to if/elsif/else but is more concise and easier to read

case value
when condition1
    # code to execute if condition1 is true
when condition2
    # code to execute if condition2 is true
else
    #code to execute if none of the conditions are true
end

age = 18
case age
when 0..12
  puts "You're a child."
when 13..18
  puts "You're a teenager."
else
  puts "You're an adult."
end

0..12 and 12..18 are ranges
These are used to check if a value falls within a certain range. In this example, the first when condition checks if the age is between 0 and 12, the second when condition checks if the age is between 13 and 18. The else block is executed if none of the conditions are true. We can have as many when conditions as we need. 

# Loops
while loop: repeats a block of code as long as a specified condition is true; the condition is evaluated before each iteration, and if it's false the loop is terminated. The 'end' keyword marks the end of the loop or block.

i++ is not part of the syntax in Ruby

while condition
    # code to execute
end

i = 0
while i < 5
  puts i
  i += 1
end # Output: 0 1 2 3 4

until loop: repeats a block of code as long as a specified condtion is false

until condition
    # code to execute
end

i = 0
until i >= 5
  puts i
  i += 1
end # Output: 0 1 2 3 4

for loop: repeats a block of code for a specified number of times

for variable in range
    # code to execute
end

for i in 0..4
  puts i
end # Output: 0 1 2 3 4

each loop: repeats a block of code for each element in a collection

collection.each do |variable|
    # code to execute
end

[1, 2, 3, 4, 5].each do |i|
  puts i
end # Output: 1 2 3 4 5

[1, 2, 3, 4, 5].each do |num|
  puts num
end

# Built-in methods in Ruby
puts and print : print data to the terminal (puts creates a new line, print doesn't)
puts "Hello, world!"
print "Hello, world!"

String methods

length : returns the length
"Hello, world!".length  # => 13
strip : returns a copy of the string with leading and trailing white space removed
" Hello, world! ".strip  # => "Hello, world!"
split : splits the string into a array of substrings based ona delimiter (like a space or comma)
"Hello, world!".split(",")  # => ["Hello", " world!"]
start_with? : checks if the string starts with a specified substring
"Hello, world!".start_with?("Hello")  # => true
end_with? : checks if the string ends with a specified substring
"Hello, world!".end_with?("world!")  # => true
include? : checks if the string contains a specified substring
"Hello, world!".include?("world")  # => true
upcase : returns a copy with all lowercase letters replaced with uppercase
"Hello, world!".upcase  # => "HELLO, WORLD!"
downcase : returns a copy with all uppercase letters replaces with lowercase
"Hello, world!".downcase  # => "hello, world!"
capitalize : returns a copy with the first character converted to uppercase and the remaining to lowercase
"hello, world!".capitalize  # => "Hello, world!"
gsub : returns a copy with all occurences of a pattern replaces with another string
"Hello, world!".gsub("world", "Ruby")  # => "Hello, Ruby!"
to_i : converts the string to an integer
"10".to_i  # => 10
to_f : converts the string to a floating-point number
"10.5".to_f  # => 10.5

concatinating strings
str1 << str2
str1.concat(str2)

(Using the shovel operator << - can also push onto an array)


Number methods

abs : returns the absolute value of a number
-10.abs  # => 10
round : rounds a floating-point number to the nearest integer
10.5.round  # => 11
floor : returns the largest integer less than or equal to a number
10.5.floor  # => 10
ceil : returns the smallest interger greater than or equal to a number
10.5.ceil  # => 11

Array methods

length : returns the length of the array
[1, 2, 3, 4, 5].length  # => 5
push : adds an element to the end of the array
[1, 2, 3, 4, 5].push(6)  # => [1, 2, 3, 4, 5, 6]
pop : removes the last element of the array
[1, 2, 3, 4, 5].pop  # => [1, 2, 3, 4]
first : returns the first element of the array
[1, 2, 3, 4, 5].first  # => 1
last : returns the last element of the array
[1, 2, 3, 4, 5].last  # => 5
join : joins all elements of the array into a string
[1, 2, 3, 4, 5].join  # => "12345"
index : returns the index of the first element equal to a specified value
[1, 2, 3, 4, 5].index(3)  # => 2
reverse : returns a new array with the elements in reverse order
[1, 2, 3, 4, 5].reverse  # => [5, 4, 3, 2, 1]
sort : returns a new array with the elements sorted
[5, 3, 1, 2, 4].sort  # => [1, 2, 3, 4, 5]
include? : checks if the array contains a specified element
[1, 2, 3, 4, 5].include?(3)  # => true

Array methods with blocks
Blocks are used to group statements together and pass them to methods as arguments. Blocks are enclosed in curly braces or do/end keywords.
[1, 2, 3, 4, 5].each do |i|
  puts i
end # output: 1 2 3 4 5

The each method iterates over each element and executes the code block for each. 
i represents the current element of the array.

Hash methods

length : returns the number of key-value pairs in the hash
{ "name" => "Alice", "age" => 30 }.length  # => 2
has_key? : checks if the hash contains a specified key
{ "name" => "Alice", "age" => 30 }.has_key?("name")  # => true
has_value? : checks if the hash contains a specified value
{ "name" => "Alice", "age" => 30 }.has_value?(30)  # => true
keys : returns a new array with all the keys of the hash
{ "name" => "Alice", "age" => 30 }.keys  # => ["name", "age"]
values : returns a new array with all the values of the hash
{ "name" => "Alice", "age" => 30 }.values  # => ["Alice", 30]
empty? : checks if the hash is empty
{ "name" => "Alice", "age" => 30 }.empty?  # => false


# Basic Methods
Methods group statements together and give them a name, allowing them to be called multiple times throughout a program.
Methods are a set of instructions that can be called on an object; reusable blocks of code that perform a specific task.

def say_hello
  puts "Hello, world!"
end

The def keyword is used to define a method. The code block is executed when the method is called.

Methods are called using the method name followed by parentheses; if the method takes arguments, they are passed inside the parentheses.
say_hello() # => Hello, world!

def say_hello(name)
  puts "Hello, #{name}!"
end

say_hello("Alice") # => Hello, Alice!

Methods can return values to the called using the return keyword. The return value is the result of the method's execution. If a return keyword is not used, the method will return the last evaluated expression.

def add(a, b)
  return a + b
end

add(1, 2) # => 3


# Testing with RSpec
Testing ensures correctness, facilitates refactoring, acts as documentation, improves design, saves time, improves collaboration, boosts confidence





# notes from class

Ruby is to Rails what JS is to Angular
Ruby is the language and Rails is the framework


numbers = [1, 2, 3, 4, 5, -1, -2, -3, -4, -5, 0, 0, 0, 0, 0]

sum = 0
positive_count = 0
negative_count = 0
zero_count = 0

numbers.each do |num|
  sum += num

  if num > 0
    positive_count += 1
  elsif num < 0
    negative_count += 1
  else
    zero_count += 1

  puts "Sum: #{sum}"
end


def count_characters_from_each_word(str)
  words = str.split(' ')
  result = {}
  words.each do |word|
    result[word] = word.length
  end
  return result
end

puts count_characters_from_each_word("Hello world!")



# gem file
similar to a package file
Gemfile is similar to packge.json

Inside the Gemfile:
gem "rspec"

bundle install 
This will install dependencies listed in the Gemfile

Installing rspec with create a spec folder
Then we can create a file app_spec.rb

In that file (the test file):

require_relative '../app'

describe '#count_characters_from_each_word' do
  it 'returns a hash with the count of characters from each word' do
    expect(count_characters_from_each_word('Hello world')).to eql?({"Hello"=>5, "world"=>5})
  end
end



to run all the tests:
rspec

to run one file at a time:
rspec spec/exercise_1_spec.rb


After forking the Repl from German's link:
in the shell, run bundle install


# How to put Repls on GitHub
Under Tools:
Git
Click on initialize repository
Add a remote repository in the settings
Connect to GitHub
Login
only select repositories (or all repositories)

Create a new repo on GitHub
Then run the commands to connect the Repl to the repo
