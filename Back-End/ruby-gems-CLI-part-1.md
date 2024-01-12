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
