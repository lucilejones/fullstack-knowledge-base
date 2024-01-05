# Online code editor and IDE
Replit
https://replit.com/

(We will eventually install Ruby on our computers.)

Create a new Replit by clicking on "Create repl"
Choose Ruby as the template
Click the "Run" button to run code

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