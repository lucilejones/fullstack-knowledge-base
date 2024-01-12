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