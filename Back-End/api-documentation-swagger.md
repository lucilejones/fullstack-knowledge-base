# Introduction
API documentation gives guidance on how to use an API.
-for frontend developers to know how to integrate the API into their apps
-for new developers of the API
-a reference for what the API does and how it works

The development life cycle:
Design the API
Develop the API
Test the API
Document the API
Deploy the API
Monitor the API
Maintain the API
Update the API
Retire the API
Repeat

Documentation can be written in different formats, like Markdown, HTML, or PDF.
One popular format is Swagger.

# Swagger
A set of open-source tools built around the OpenAPI specification that can help us design, build, document, and consuem REST APIs.

We'll be using an integration with swagger called rswag.
Rswag is a tool that lets us write Swagger documentation as integration test in Ruby on Rails.
-will provide a UI to view the documentation and also a way to test the API in the browser.

# Setup Swagger in Ruby on Rails
Create a new rails api

rails new swagger_tutorial --api

Add the following gems to the Gemfile:
gem 'rswag'
gem 'respec-rails', '~> 6.1.0'

run bundle install

Then run rails generate rspec:install
Then run rails g rswag:install

