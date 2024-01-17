# Bcrypt and passwords
Authentication - verifying a user is who they say they are. Usually asking for a name and password.

We will use the Bcrypt gem to encrypt passwords.
Bcrypt is a hashing algorithm; it is a one-way encryption algorithm, meaning that once a password is encrypted it cannot be decrypted. We can just try a password and see if it matches the encrypted password.

Add Bcrypt to the Gemfile
gem "bcrypt"
Then run bundle install

An example of how to use Bcrypt (in a main.rb file):
require "bcrypt"

my_password = BCrypt::Password.create("my password")

puts my_password

Then we click run. We can't manually execute main.rb since in the context of that file, we don't have access to the gems. We have to run it from the repl.

The bcrypt gem provides a BCrypt::Password class that we can use to create a new password.
The BCrypt::Password.create method takes a string as an argument and returns a new password object. That object has a to_s method that returns the encrypted password as a string.
If we run the file, we'll see output like this:
$2a$12$Jn9OrCsfhWp9JosMmlGyoeTkq/8.YyNKdx6RPo32uyEbUI7I8UZKa

The string starts with $2a$12$ and ends with a string of characters. 
The $2a$12$ is called the salt and is used to make the password more secure. It is generated randomly and is different every time we run the BCrypt::Password.create method, and is stored with the encrypted password so the password can be verified later.

The encrypted password can be verified using the == method. This method takes a string as an argument and returns true if the string matches the encrypted password and returns false if it doesn't.

Example:
require "bcrypt"

my_password = BCrypt::Password.create("my password")

puts my_password == "my password" # true

puts my_password == "not my password" # false

An example of how to use the == method to verify a user's password:
require "bcrypt"

class User
  attr_accessor :username, :password

  @@users = []
  def initialize(username, password)
    @username = username
    @password = BCrypt::Password.create(password)
    @@users << self
  end

  def self.authenticate(username, password)
    user = User.find_by_username(username)

    if user && user.password == password
      return user
    else
      return nil
    end
  end

  def self.all
    @@users
  end

  def self.find_by_username(username)
    user = all.find do |user|
      user.username == username
    end
    user
  end
end

User.new("username", "password")
user = User.find_by_username("username") # returns user object
puts User.authenticate("username", "password") # returns user object
puts User.authenticate("username", "not password") # returns nil

The User.authenticate method takes a username and password as arguments and returns a user object if the username and password match a user in thedatabase. The User.authenticate method uses the == method to verify the password entered matches the one stored.


# Incorporating BCrypt into Countries of the World CLI
We'll use BCrypt to encrypt the user's password when they create an account and then to verify the user's password when they log in.

Add gem 'bcrypt' to the Gemfile and run bundle install

Then we'll create a User class in its own file lib/user.rb:
require "bcrypt"

class User
  attr_accessor :username, :password

  @@users = []
  def initialize(username, password)
    @username = username
    @password = BCrypt::Password.create(password)
    @@users << self
  end

  def self.authenticate(username, password)
    user = User.find_by_username(username)

    if user && user.password == password
      return user
    else
      return nil
    end
  end

  def self.all
    @@users
  end

  def self.find_by_username(username)
    user = all.find do |user|
      user.username == username
    end
    user
  end
end

And in the cli.rb we need to update our start method, add a call to the authenticate method:
If the user cannot be authenticated, we won't allow them to search for a country.
We'll also add a call to the create_account method if the user chooses to sign up.

require_relative "./scraper.rb"
require_relative "./user.rb"

 . . .

  def start
    Scraper.scrape_countries
    puts 'Welcome to the Countries of the World CLI!'
    authenticate
    puts 'Please enter a country name to get more information about it.'
    input = gets.strip
    country = Country.all.find { |country| country.name.downcase == input.downcase }
    if country.nil?
      puts 'Sorry, that country is not in our database. Please try again.'
    else
      puts "Name: #{country.name}"
      puts "Capital: #{country.capital}"
      puts "Population: #{country.population}"
      puts "Area: #{country.area}"
    end
  end

 . . .

comment # authenticate user or create account
  def authenticate
    authenticated = false

    until authenticated
      puts 'Please login or sign up'
      puts 'Which do you choose? (sign up/login)'

      if get_input == 'login'
        authenticated = login
      else
        create_account
      end
    end
  end

  comment # check if user is in User class and if password matches
  def login
    puts 'Please enter your username:'
    username = gets.chomp
    puts 'Please enter your password:'
    password = gets.chomp
    result = User.authenticate(username, password)

    if result
      puts "Welcome back #{username}!"
    else
      puts 'Sorry, that username and password combination is not recognized. Please try again.'
    end
    result
  end

  comment # create a new user and add to User class
  def create_account
    puts 'Please enter a username:'
    username = gets.chomp

    puts 'Please enter a password:'
    password = gets.chomp

    User.new(username, password)
    puts 'Account created'
  end

end


# File and Data Manipulation
Changing the contents of a file or data are very common tasks in programming. In special cases, we may need to monitor outputs of responses and bundle them into a file. Or we may need to extract a csv file from a database and convert it to a json file. 

We will use the Countries of the World CLI for an example of how to add onto a file and manipulate data. We'll add usernames and their encrypted hashes into an external json file. The json file will store the data in a key-value pair format. 
(In the real world, we'd use a database to store this info, but for the example we'll use a json file.)

In our user.rb file, we'll define a method that stores the user info.
We need to require 'json' in the user.rb file in order to use the JSON class:

require 'json'
.
.
.
  def store_credentials(user)
    file_path = 'users.json'

    unless File.exist?(file_path)
      File.open(file_path, 'w') { |file| file.write(JSON.generate([])) }
    end
  end

The File class is used to read and write files. 
The File.exist? method takes a file path as an argument and returns true if the file exits and false if it doesn't.
The File.open method takes a file path and a block as arguments and opens the file for reading and writing.
The JSON.generate method takes an array as an argument and returns a JSON string. This will create a JSON file with an empty arry if the file doesn't exist.

def store_credentials(user)
    file_path = 'users.json'

    unless File.exist?(file_path)
      File.open(file_path, 'w') { |file| file.write(JSON.generate([])) }
    end

    file = File.read(file_path)
    users_data = JSON.parse(file)

    users_data << { 'username' => user.username, 'password' => user.password }

    File.open(file_path, 'w') { |file| file.write(JSON.generate(users_data)) }
  end

Then we call store_credentials as the user is initialized:

def initialize(username, password)
    @username = username
    @password = BCrypt::Password.create(password)
    store_credentials(self)
    @@users << self
  end

Once we check to see whether the file exists, we will read the file and parse the JSON string into an array. We then add the user's username and password to the array and write the array back to the file.

Now we can sign up and add the credentials to users.json.

This will create a small issue with login.
Every time we load the file and initialize new user instances we are creating a new password each and every time. This is not correct. We need to change our logic. To fix this, we can add a new paramenter to indicate if the password is a pre-existing hash or a new one.

def initialize(username, password, password_pre_hashed = false)
    @username = username
    @password = password_pre_hashed ? BCrypt::Password.new(password) : BCrypt::Password.create(password)
    @@users << self
  end

We use a ternary operator to check if the password is pre-hashed. If it is, we use the BCrypt::Password.new method to create a new password object from the pre-hashed password. We aren't able to verify the password with the == method if it's pre-hashed. 
The BCrypt::Password.new method takes a string as an argument and returns a password object. 
If the password is not pre-hashed, we use the BCrypt::Password.create method to create a new password object from the password.

We can add true for the new argument in our load_users_from_file method because lodaing users will ahve an existing password pre-hashed.

def self.load_users_from_file
    file_path = 'users.json' # Path to your JSON file

    if File.exist?(file_path)
      file = File.read(file_path)
      users_data = JSON.parse(file)

      users_data.each do |user_data|
        User.new(user_data['username'], user_data['password'], true) # true indicates that the password is already hashed
      end
    end
  end

Because we will store credentials in the CLI class, we need to change it from an instance method to a class method (add def self.store_credentials instead of def store_credentials.)
Also take out the method call in the initialize method for the User class:
store_credentials(self)

Then we need to include the argument for when a user creates an account.
In our cli.rb file:
def create_account
    puts 'Please enter a username:'
    username = gets.chomp

    puts 'Please enter a password:'
    password = gets.chomp

    user = User.new(username, password, false) # false indicates that the password is not hashed
    User.store_credentials(user) #
    puts 'Account created'
  end

And load the users from file at the start:
class CLI
  def start
    User.load_users_from_file
    Scraper.scrape_countries

This will actually create the file user.json with an array populated with users.
example: [{"username":"user1","password":"$2a$12$S9TGuHbnjSwHzoqcLhZ9SuSbpUmYukhKLjQT6hHMev0EXa0HV/uQS"}]


# Video: USA Covid 19 CLI - Authentication
We'll start with a User class. In a user.rb file (in the lib folder):
require 'bcrypt'

class User
    attr_accessor :username, :password

    @@users = []

    def initialize(username, password)
        @username = username
        @password = BCrypt::Password.create(password)

        @@users << self
    end

    def.self.all
        @@users
    end

end

We want to store all the users as they're created in the array @@users. The passwords are being hashed before they're stored.

We want to add a method to authenticate the user:
def self.authenticate(username, password)
    user = User.find_by_username(username)
end

We start by finding the user (by username). We'll define a separate method for that (using the built-in method find):
def self.find_by_username(username)
    user = all.find do |user|
        user.username == username
    end
end

We compare each user's username to the username we pass in as an argument. The method will return a user. If we can't find a user, it's going to return nil.

Then in the authenticate function we'll say, if user is not nil and that user's password equals the password that we passed in, we want to return that user. Otherwise we want to return nil. 

if user && user.password == password
    return user
else
    return nil
end

Full method:
def self.authenticate(username, password)
    user = User.find_by_username(username)

    if user && user.password == password
        return user
    else
        return nil
    end
end


Next, in the cli.rb file we want to authenticate the user.
In the run method, before even scraping the data, we'll add a call to an authenticate method.
def run
    authenticate
    Scraper.scrape_data
...
end


At the end of the cli.rb file (still inside the CLI class):

def authenticate
    authenticated = false

    until authenticated
        puts "Please login or sign up"
        puts "Which do you choose? (sign up/login)"

        get_input = gets.chomp
        if get_input == 'login'

        else

        end

    end
end

We'll first check to see if the user is going to login or signup. 
We'll define a variable authenticated and first set it to false. We don't want them to go into the program unless authenticated is true.

We can define another method called login and call the authenticate method from the User class. (So we need to require_relative the user.rb file in the cli.rb file.) Because User.authenticate returns a value, we can save it in a result. Then if the user is authenticated, result is true, we'll puts a welcome message. Otherwise, we'll puts "Invalid username or password". 

def login
    puts "Please enter your username"
    username = gets.chomp
    puts "Please enter your password"
    password = gets.chomp

    result = User.authenticate(username, password)

    if result
        puts "Welcome back #{username}"
    else
        puts "Invalid username or password"
    end

    result
end

And then we'll return result. We want to return result because we want to set our authenticated variable to login.

if get_input == login
    authenticated = login
else

end

If the user wasn't authenticated becase the user is nil, then athenticated will be nil and that method will keep on looping.

We also need a create_account method (under the login method in the cli.rb):

def create_account
    puts "Please enter your username"
    username = gets.chomp
    puts "Please enter your password"
    password = gets.chomp

    User.new(username, password)
    puts "Account created!"
end

We'll call create_account in our authenticate method.

def authenticate
    authenticated = false

    until authenticated
        puts "Please login or sign up"
        puts "Which do you choose? (sign up/login)"

        get_input = gets.chomp
        if get_input == 'login'
            authenticated = login
        else
            create_account
        end

    end
end


# Video: USA Covid-19 CLI - Adding Users to File
Adding users to an external file as they signup.
We can use Ruby's built-in File class.
We're going to save the data in a JSON file, which is like a JSON object, sort of like a hash. Then we can access info through keys. 

In the user.rb file, in the initialize method, we'll call a User.store_credentials method and pass in self.

def initialize(username, password)
    @username = username
    @password = BCrypt::Password.create(password)

    User.store_credentials(self)
    @@users << self
end

Then we'll define a class method (still in the class User, but at the end):

def self.store_credentials(user)
    file_path = "users.json"
    unless File.exist?(file_path)
        File.open(file_path, "w") { |file| file.write(JSON.generate([])) }
    end
end

This method will create a file if it doesn't exist and then add a user to the file.

We'll declare a variable to represent out file path. And if the file doesn't exist already, we'll create it. 
Then we use the File.open method to open a file with that file path - it's going to create that file (if it isn't already created) - and the second argument is a string that represents read or write access. "w" write indicates both.
Then we include a one-line expression; as we're accessing the file once it's created, we want to write to the file a JS value as an empty array. (Because we're going to have an array of users.)

In order to turn Ruby values into a JSON object we need to require 'json' at the top of the user.rb file.

At this point so far if we run the program and sign up a new user we'll get a users.json file with an empty array. Then we can add the user to the file.

We start by reading the file, File.read(file_path), and then we want to get the file, grab its content and parse that JSON object into a Ruby array.
user_data = JSON.parse(file)
Then we can use the shovel operator to push the user to the Ruby array. We won't just push the user because we want to store a hash and not an instance. So we have to manually define the hash.
Ultimately we want an array of hashes.
Next we open the file, and write to it the array of users_data. (Right now, with "w" this is going to override the file. If we wanted to append, we could use "a" and write different logic. Though that wouldn't work in this example.)



def self.store_credentials(user)
    file_path = "users.json"
    unless File.exist?(file_path)
        File.open(file_path, "w") { |file| file.write(JSON.generate([])) }
    end

    file = File.read(file_path)
    users_data = JSON.parse(file)

    users_data << { username: user.username, password: user.password }

    File.open(file_path, "w") { |file| file.write(JSON.generate(users_data)) }
end


# Video - USA Covid 19 CLI - Loading Users from File
We want to load in the file of users at the start of the program so users can login.

In the run method (in the cli.rb file), right before we authenticate and scrape the data, we'll make a call to the method User.load_users_from_file

def run
    User.load_users_from_file
    authenticate
...
end

We'll define that method in the user.rb file:

def self.load_users_from_file
    file_path = 'users.json'

    if File.exist?(file_path)
        file = File.read(file_path)
        users_data = JSON.parse(file)

        users_data.each do |user_data|
            User.new(user_data['username'], user_data['password'])
        end
    end

end

We first read the file, then use Ruby to parse the JSON file, then iterate and create user instances (using User.new) from the array of hashes.

The issue here, though, is that when we use the password from the hash to create the user instance it will re-encrypt the hash into something else. 
(In the initialize method in the User class: @password = BCrypt::Password.create(password))
But we want to reuse the hash. 
What we can do is include an additional parameter to the initialize method indicating whether it is an existing password. We'll set the default to false.
def initialize(username, password, exisiting_hash = false)
    @username = username
    @password = existing_hash ? BCrypt::Password.new(password) : BCrypt::Password.create(password)

    @@users << self
end

If existing_hash is true, we can use BCrypt to create a new instance or a new value from the existing hash; we're not re-encrypting it, we're just creating a new hash (that we can still use with the existing password the user entered). 
But then we want to take out the call to User.store_credentials(self)
We don't want to add those same users into the file (the same one that we're loading at the beginning).

Instead, we can add the user to the file when they sign up for an account, right after the user instance has been made.

In the cli.rb file:
def create_account
...
    user = User.new(username, password)
    User.store_credentials(user)
    puts "Account created!"
end

Then in the self.load_users_from_file (in the user.rb file), we want to set the third parameter to true:

users_data.each do |user_data|
    User.new(user_data['username'], user_data['password'], true)
end

