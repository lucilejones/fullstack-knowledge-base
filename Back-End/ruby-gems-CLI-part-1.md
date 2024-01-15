# Ruby Gems
Ruby gems are packages of Ruby code used to extend or modify the functionality of Ruby applications.
Often shared through the RubyGems platform: RubyGems.org

Examples:
Nokogiri, Pry, HTTParty, Sinatra, Rails, Rake, RSpec, Rubocop, Faker

# Using a Gem
To use the Nokogiri gem to parse HTML and XML documents, we need to add to the gemfile:
source "https://rubygems.org"
ruby '3.1.2'
gem "nokogiri"

We use the source method to specify the source fo the gems we want to use.
Then we use the gem method to specify the gem we want to use.
We can add multiple lines:
gem 'gem-1'
gem 'gem-2'

We can also specify the version we want to use:

source "https://rubygems.org"

git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

ruby '3.1.2'
gem "nokogiri", "1.15.4"

1 is the major version, 15 is the minor version, 4 is the patch version

If we don't specify the version, it will use the latest version of the gem.

After we add the gem to our Gemfile we need to run
bundle install
This will install the gem and all of its dependencies. (The equivalent of npm install for JS projects.) This will just be installed for this project.
To install a gem globally, we have to run
gen install gem-name 

The Gemfile.lock file keeps track of the gems we're using in our project.

...
DEPENDENCIES
  nokogiri (= 1.15.4)
...

To practice with Nokogiri, we'll create an html file: index.html
Then we can extract the text from the h1 element.
In the main.rb file:

require "nokogiri"

doc = Nokogiri::HTML(File.open("index.html"))

puts doc.css("h1").text

We use the require method to require the Nokogiri gem and use its provided objects.
The Nokogiri::HTML module is used to parse the HTML document.
The File.open method is used to open the index.html file. We pass this file to the module, and it will parse the HTML document and store it in the doc variable. This means it will convert the HTML document into a format we can use. 

Then we use the css method (a nokogiri method) to select the h1 element.
Then we use the text method (another nokogiri method) to get the text or content of the h1 element.

When we execute the main.rb file, it will print Hello World to the terminal.


# Web scraping with Nokogiri and httparty
We can use the Nokogiri gem and the httparty gem to make HTTP requests and parse the HTML and XML documents. This is a form of web scraping.

Define a class called API and include a class method called get_films_by_year that takes in a year as an argument. We will use year to get a list of films from Wikipedia.

Then use the HTTParty.get method to make a GET request to the Wikipedia page for the year we pass in as an argument.

We can't make use of the unpartsed page until we parse it into a nokogiri object.

Then we extract the elements as nokogiri instances.

Targeting elements goes from left to right in the string as we go down the hierarchy. Parent element to child element.
The css method selects the table.wikitable.sortable tr td:nth-child(2) i a element.
This element represents the series of hierarchy levesl to get to the table and the elements. We need to inspect the elements on the webpage and find the ones we want based off the selector or class name. This takes trial and error. 
Basically, we're finding the parent element, targeting it by its CSS and then finding the child element and targeting it.

In this example, first we target a table element with classes wikitable and sortable.
Then we target the tr element (since it will include all the rows of the films), and we need to specify that we want td for the table data (but to remove the header rows).

Then we have the nth-child(2) element. This allows us to extract the second child element from every td. 
Then we have an i element.
And then an a element.

This will return an array of nokogiri instances. 

Then we can use that to iterate and strip the text from the elements.

Then we use the each_with_index method to iterate over the films and print out the films to the terminal. 

The result of this code is a list of films from 2019 once you execute it.


Nokogiri can be boggy and difficult to use. It is not always accurate. 
Our code can break if the website changes, if the website it down, if the website is slow, or if we're making a lot of requests. 
In a practical sense, we're more likely to get data from a website using an API rather than web scraping. 


# Countries of the World CLI
A CLI is a command line interface - a program that allows you to interact with your computer using the command line. (Example: the Angular CLI)

In the lib directory, in a file called cli.rb:
class CLI
  def start
    puts "Welcome to the Countries of the World CLI!"
    puts "What is your name?"
    name = gets.strip
    puts "Hello #{name}!"
  end

  def get_input
    gets.strip
  end
end

To start, we define a class CLI with methods. First we print a welcome message and ask the user for their name.

Then we use the gets global method to get the user's input and use strip to remove any whitespace. We will use puts to print the user's name. 

We'll create a new instance of the CLI class in the main.rb and call the start method:

require_relative './lib/cli.rb'

CLI.new.start


To add tests:
run
bundle add rspec
This will add respc to our Gemfile and lock file.
run
bundle exec rspec --init
This initializes rspec into the project folder. It creates an .rspec filr and a spec directory with a spec_helper.rb file.
Then we run
gem install rspec
This lets us use the rspec gem in the shell (commands like rspec)

Then we create a new file spec/cli_spec.rb with the following test code:
require_relative "../lib/cli.rb"

RSpec.describe CLI do
  describe "#start" do
    it "prints a welcome message and asks the user for their name" do
      cli = CLI.new

      # Stubbing the standard input to simulate user input
      allow(cli).to receive(:gets).and_return("John Doe\n")

      # Expecting specific output to standard output
      expect { cli.start }.to output(
        "Welcome to the Countries of the World CLI!\nWhat is your name?\nHello John Doe!\n"
      ).to_stdout
    end
  end
end

RSpec.describe method - we describe the start method of what we will be testing
it method - describe what the start method does
The test demonstrates the creation of a new CLI object
allow method - allows the cli objec to receive the get_input method
expect method - expect the cli.start method to output the welcome message and ask the user for their name

run
bundle exec rspec
to run the test

rspec - will use the globally installed version of rspec
bundle exec rspec - will use the locally installed version of rspec


# Web Scraping with Nokogiri and httparty Part 2
website that allows legal web scraping: www.scrapethissite.com
Some do not allow it and you could get banned

We'll add nokogiri and httparty to the Gemfile, then run bundle install

In a new file scraper.rb
require "nokogiri"
require "httparty"

module Scraper
  INDEX_URL = 'https://www.scrapethissite.com/pages/simple/'
  def self.scrape_countries
    unparsed_page = HTTParty.get(INDEX_URL)
    parsed_page = Nokogiri::HTML(unparsed_page.body)
    puts parsed_page
  end
end


The Scraper module defines a scrape_countries method that makes a GET request to the INDEX_URL and parses the HTML document.
The HTTParty.get method makes a GET request to the url.
The Nokogiri::HTML method parses the HTML document.
The puts method prints out the parsed HTML document to the terminal.

Then we'll call the scrape_countries method from the start method in the cli.rb file.
require_relative "scraper.rb"

class CLI
  def start
    Scraper.scrape_coutnries
...

We need to find a way to select the div element that holds all the country elements and then select the country elements. They all have the same class, country, so we can use that.
countries = parsed_page.css("div.country")
puts countries
This will give us the divs with the class of coutry. But we want to select the country name, capital, population, and area. 

    countries.each do |country|
      name = country.css("country-name").text
      capital = country.css("country-capital").text
      population = country.css("country-population").text
      area = country.css("country-area").text

      puts "#{name} #{capital} #{population} #{area}"

However, this will get all blanks. We must've done something wrong.
We can use the byebug gem.

The byehub gem helps us debug our code. It's similar to the debugger in JS. We can pause the code and inspect it.

Add gem "byebug" to the Gemfile.
Then run bundle install.

We can add the byebug gem to our scraper.rb file.
require "byebug"

Then we add debugger where we want the code to pause. We'll put it right after
area = country.css("country-area").text
debugger;


This will pause our code and we have access to the variables. In the terminal we see:
[16, 25] in /home/runner/countriesoftheworldcli/lib/scraper.rb
   16:     countries.each do |country|
   17:       name = country.css("country-name").text
   18:       capital = country.css("country-capital").text
   19:       population = country.css("country-population").text
   20:       area = country.css("country-area").text
   21:       debugger;
=> 22:       puts "#{name} #{capital} #{population} #{area}"
   23:     end
   24:   end
   25: end

If we type name and print enter, it will return an empty string: ""
This is where we can try different things with the css selectors to actually get what we want. 
If we inspect the webpage, we see that the country class element has an h3 and a div with the country info.
This means country represents both those elements as well. We need to select the h3 and div separately.

If we enter country in the terminal, we get back a Nokogiri instance. 
We can call nokogiri methods on it.
So we can type country.css("h3").text

We do get back the country name, but it a weird format:
"\n                            \n                            Andorra\n                        "

So we can use the strip method to remove the whitespace:
country.css("h3").text.strip
That prints "Andorra"

The css selectors should also have periods at the beginning; it should be .country-name instead of country-name, etc. That's because these are classes and shouldn't be selected as elements. 

We enter exit in the terminal to exit the debugger, 
and then we can updated the scraper.rb file with periods in the css selectors.
require "nokogiri"
require "httparty"
require "byebug"
module Scraper
  INDEX_URL = 'https://www.scrapethissite.com/pages/simple/'
  def self.scrape_countries
    unparsed_page = HTTParty.get(INDEX_URL)
    parsed_page = Nokogiri::HTML(unparsed_page.body)
    countries = parsed_page.css("div.country")

    countries.each do |country|
      name = country.css(".country-name").text.strip
      capital = country.css(".country-capital").text.strip
      population = country.css(".country-population").text.strip
      area = country.css(".country-area").text.strip
      puts "#{name} #{capital} #{population} #{area}"
    end
  end
end

We use .strip to take out the whitespace.
We also need to comment our the debugger line in our code.

Then we get a list of countries printed to the terminal!

We can now use this data for our CLI.

We can create a country class to store the data. 
In a new file called country.rb:
class Country
  attr_accessor :name, :capital, :population, :area

  @@all = []

  def initialize(name, capital, population, area)
    @name = name
    @capital = capital
    @population = population
    @area = area
    @@all << self
  end

  def self.all
    @@all
  end
end

We use the attr_accessor method to create getters and setters for the name, capital, population, and area attributes.
The @@all class variable stores all the instances of the Country class.
The initialize method initializes a new Country object with a name, capital, population, and area.

We can update the scraper.rb file to create a new Country object and store the data in it.

require_relative "./country.rb"
...
    countries.each do |country|
      name = country.css(".country-name").text.strip
      capital = country.css(".country-capital").text.strip
      population = country.css(".country-population").text.strip
      area = country.css(".country-area").text.strip
      Country.new(name, capital, population, area)
    end
...

Then let's add search functionality and display the data to the user.
In the cli.rb file:
require_relative "./scraper.rb"

class CLI
  def start
    Scraper.scrape_countries
    puts "Welcome to the Countries of the World CLI!"
    puts "What is your name?"
    name = gets.strip
    puts "Hello #{name}!"
    puts "Please enter a country name to get more information about it."
    input = gets.strip
    country = Country.all.find { |country| country.name.downcase == input.downcase }
    if country === nil
      puts "Sorry, that country is not in our database. Please try again."
    else
      puts "Name: #{country.name}"
      puts "Capital: #{country.capital}"
      puts "Population: #{country.population}"
      puts "Area: #{country.area}"
    end
  end

  def get_input
    gets.strip
  end
end


# Adding technical documentation
Can be written text or illustrations that accompany computer software or are embedded in the source code. It can explain how the software operates or how to use it.

Internal documentation: written for the developer; not meant for the end user. For the developer to understand the codebase and the architecture of the codebase.
External documentation: written for the end user. For the user to understand how to use the software.

In the readme.md file we can add:
# Countries of the World CLI

## Description

The Countries of the World CLI is a command line interface that allows you to get information about countries. It uses this [site](https://www.scrapethissite.com/pages/simple/) to get the information.

## Installation

1. Clone the repository
2. Run `bundle install`
3. Click `run`to run project.

## Gems

### Nokogiri

[Nokogiri](https://rubygems.org/gems/nokogiri) is a gem that allows you to parse HTML and XML documents.

### HTTParty

[HTTParty](https://rubygems.org/gems/httparty) is a gem that allows you to make HTTP requests.

### RSpec

[RSpec](https://rubygems.org/gems/rspec) is a gem that allows you to test your code.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).




# video: USA COVID-19 CLI - Setup
url: worldometers.info/coronavirus/country/us/

Make a directory called lib
Then we have a file called cli.rb
In that file we define a class with methods

class CLI
  def run
    greeting
    while menu != 'exit'
    end
    end_program
  end

  def greeting
    puts "Welcome to the USA Covid 19 Tracker"
  end

  def end_program
    puts "Thank you for using the USA Covid 19 Tracker"
  end

  def menu
    list_options
    input = gets.chomp.downcase
    choose_option(input)
    input
  end

  def list_options
    puts "Please select an option:"
    puts "1. List all states"
    puts "2. List top ten states with the most confirmed covid cases"
    puts "3. Print USA information"
    puts "Exit the program by entering 'exit'"
  end

  def choose_option(option)
    case option
    when "1"
      puts "Listing all states..."
    
    when "2"
      puts "Listing top ten states with the most confirmed covid cases..."
    end

    when "3"
      puts "Printing USA information..."
    end
  end

end

In order to access these methods in our program, we need to require the cli.rb file in our main.rb file, then we'll create an instance of the class and call the run method:

require_relative "./lib/cli.rb"

CLI.new.run

Then we define our other methods and add an input to the menu method.
Then we call choose_option and take the input.

If we want to show the list of options until a user enters 'exit' we can return the input in the menu method and then put a while loop in the run method.
while menu != 'exit'


# video: USA Covid-19 CLI - Web Scraping USA Info
We create a file called scraper.rb (in the lib directory) with a module Scraper

module Scraper
  URL = "https://www.worldometers.info/coronavirus/country/us/"

  def self.extract_usa_data (doc)
    country_main = doc.css('.maincounter-number')
    usa_confirmed_cases = country_main[0].text.strip
    usa_deaths = country_main[1].text.strip
    usa_recovered = country_main[2].text.strip

    Country.new("USA", usa_confirmed_cases, usa_deaths, usa_recovered)
  end

  def self.extract_states_data

  end

  def self.scrape_data
    puts "Scraping data..."
    unparsed_page = URI.open(URL)
    doc = Nokogiri::HTML(unpared_page)
    extract_usa_data(doc)
  end

end

In order to do the web scraping, we need to add to our Gemfile:

gem 'nokogiri'
gem 'open-uri'

(open-uri is similar to httparty)

Then in the shell we run bundle install

In the scraper.rb file:
require 'nokogiri'
require 'open-uri'

Then we'll grab the url and set a constant property in the module:
URL = "https://www.worldometers.info/coronavirus/country/us/"

And we'll add sending a request in the self.scrape_data method.
First we save the data in a variable unparsed_page
Then declare a variable doc which represents the parsed page.
Then we can puts doc (which will be a ton of information - more than we need).
Instead we can call extract_usa_data and pass in the doc variable.

We need to require the scraper file in our cli.rb file and run that method at the beginning of our program.

In the cli.rb file:

require_relative 'scraper.rb'

Then at the beginning of our run method we want to call Scraper.scrape_data:
  def run
    Scraper.scrape_data
    greeting
    while menu != 'exit'
    end
    end_program
  end

In order to know what to extract in the extract_usa_data method, we'll need to inspect the webpage. What we want is the element with the class "maincounter-number".
This will actually give us an array of divs that all have that class.

(At this point German includes the gem byebug in the Gemfile and then runs bundle install, then require 'byebug' in the scraper.rb file. That allows us to use a debugger.)

Then we find the specfic element of the array that we want to use:
usa_confirmed_cases = country_main[0].text.strip
(If the website ever changes its layout this code will break.)
We could add .to_i to convert it to an integer, but that will take away the commas and give us groups of numbers in sets of three, not the total 110,316,061.


Then we can create a class to format this information all in one place (the cases, deaths, and recovered numbers).
We create a new file country.rb

class Country

  attr_accessor :name, :confirmed_cases, :overall_deaths, :recoveries

  @@countries = []

  def initialize(name, confirmed_cases, overall_deaths, recoveries)
    @name = name
    @confirmed_cases = confirmed_cases
    @overall_deaths = overall_deaths
    @recoveries = recoveries
    @@countries << self
  end

  def self.all
    @@countries
  end

  def self.first
    @@countries[0]
  end

end


We'll use a class variable for the countries array.
When we create an instance of a Country, we want to initalize all the attributes.
When a new instance gets created it will also get added to the class variable, the array of countries (@@countries << self).
Then we create a class method that will return countries. 


Then in the scraper.rb we need to require_relative 'country.rb'
Then in the extract usa data we can call Country.new() to create a new instance with the info we get here.

If we save that in a variable usa we can print that and see we're getting the right instance. 
Instead we'll do it in the cli.rb file. 

Then in the Country class we can grab the first country from the countries array.
def self.first
  @@countries[0]
end

In the cli.rb file we require_relative 'country.rb'
And in the third when case we extract the usa info and print it:

when "3"
  puts "Printing USA information..."
  country = Country.first
  puts country.name
  puts country.confirmed_cases
  puts country.overall-deaths
  puts country.recoveries
end


# video: USA Covid-19 CLI - Web Scraping US States
We'll target the table of States data: tbody and tr

First, in the self.scrape_data method(in the scraper.rb file) we'll add a call to the extract_states_data(doc) method (and pass the same doc we did for the usa data).
This will give us the info in a Nokogiri instance.
Then we separate each line of the table and create an array of Nokogiri instances.

We need to include a parameter - doc - to reference the argument that we're passing.

def self.extract_states_data(doc)
  states_table = doc.css('tbody tr')

  states_table[1..51].each do |row|
    name = row.css('td')[1].text.strip
    cases = row.css('td')[2].text.strip
    deaths = row.css('td')[4].text.strip
    recoveries = row.css('td')[6].text.strip

    State.new(name, cases, deaths, recoveries)
  end
end

We'll use the css method and pass in the selectors we need to use (tbody and tr)
Then we can iterate over the states_table and get the data from each row.
We can use a range [1..51] to say that we don't want the first row [0] because that's the headers, and we only want to go through to the 50 states (so the [51] index).
Then we want to extract the td, the table data.


Then we'll create a State class in a state.rb file
We could try to inherit from the Country class, except that we don't need the array of countries (@@countries), so in this case it might be better to keep them separate.

class State
  attr_accessor :name, :confirmed_cases, :overall_deaths, :recoveries

  @@states = []

  def initialize(name, confirmed_cases, overall_deaths, recoveries)
    @name = name
    @confirmed_cases = confirmed_cases
    @overall_deaths = overall_deaths
    @recoveries = recoveries
    @@states << self
  end

  def self.all
    @@states
  end

  def self.first
    @@states[0]
  end

end

Then in the scraper.rb file we require_relative 'state.rb'
And we create a new State instance in the extract_states_data method:
State.new(name, cases, deaths, recoveries)

Then in the cli.rb file we require_relative 'state.rb'
And print all the states:

when "1"
  puts "Listing all states..."
  State.all.each do |state|
    puts '---------'
    puts "Name: #{state.name}"
    puts "Cases: #{state.confirmed_cases}"
    puts "Deaths: #{state.overall_deaths}"
    puts "Recoveries: #{state.recoveries}"
    puts '---------'
  end


# video: USA Covid-19 CLI - More Options

when '2'
  puts 'Listing top ten states with the most confirmed covid cases..."
  State.all[0..9].each_with_index do |state, i|
    puts "#{i+1} #{state.name} - #{state.confirmed_cases}"
  end


We can also add another option (in the list_options method):

  puts "4. List top ten states with the most overall deaths."

Then in the when cases: 

when '4'
  puts "Listing top ten states with the most overall deaths..."

  sort_states = State.all.sor_by{|state| state.overall_deaths}

  sort_states[0..9].each_with_index do |state, i|
    puts "#{i+1} #{state.name} - #{state.overall_deaths}"
  end


First we need to sort the array of states and then iterate and list the top 10.
We can also use a condition to make sure DC doesn't get included:

if(name != 'District of Columbia')
  State.new(name, cases, deaths, recoveries)
end

We'll also need to turn the overall_deaths strings into numbers in order to accurately compare them. 
We can use gsub (a built-in method) to strip away the commas, and then turn it into an integer.
sort_states = State.all.sor_by{|state| state.overall_deaths.gsub(/,/, '').to_i}



# Notes from class 1/15/24

system("clear") - reserved word, built-in method. "clear" will clear out the terminal.

In our class API, we write a class method using self becuase we want it to not be dependent on a specific instance of the class.
def self.find_films_by_year(year)

end

For this project, Nolan used the gems: pry, nokogiri, and open-uri
And a gem json that is built into Ruby 3.

Scraping is getting the HTML document info off a webpage.

scraped_movies = JSON.parse(doc.text)

To find out whether a site is scrapable:
check their robots.txt file by doing https://example.com/robots.txt
Or check their API or ToC (usually in the footer)

In Ruby, to get access to the key of an object, we use bracket notation:
puts "#{movie["title"]}"