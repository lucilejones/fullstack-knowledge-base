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