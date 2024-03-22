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

With rwag, the Swagger documentation is written as integration tests.
We define the structure of our API (endpoints, parameters, responses) within RSpec fils located in spec/integration.

# Setup API
First we'll make a model for users:
rails generate model User name:string email:string

We might get a deprecation warning when running the above command.
DEPRECATION WARNING: Rswag::Ui: WARNING: The method will be renamed to "openapi_endpoint" in v3.0 (called from block in <main> at /Users/germancruz/Documents/projects/swagger-tutorial/config/initializers/rswag_ui.rb:11)

To fix that, in the config/initializers/rswag_ui.rb file, we replace:
    c.swagger_endpoint '/api-docs/v1/swagger.yaml', 'API V1 Docs'
with:
    c.openapi_endpoint '/api-docs/v1/swagger.yaml', 'API V1 Docs'

Then run rails db:migrate

Add the following code to the user model:
class User < ApplicationRecord
  validates :name, presence: true
  validates :email, presence: true
end

And in the seed file:
User.create(name: "Alice", email: "alice@example.com")
User.create(name: "Bob", email: "bob@example.com")

Then run rails db:seed

Next we'll generate a controller for users:
rails generate controller Users

In the users_controller.rb file:
class UsersController < ApplicationController
  def index
    users = User.all
    render json: users
  end
end

In the routes file:
Rails.application.routes.draw do
  resources :users, only: [:index]
end

# Document index endpoint
First we'll genereate a new user_spec file to document the index endpoint:
rails generate rspec:swagger UsersController --spec_path integration

--spec_path means the file will be generated in the integration folder of spec.

Then in the users_controller_spec file:
require 'swagger_helper'

RSpec.describe 'users', type: :request do
  path '/users' do

    get 'Retrieves all users' do
      tags 'Users'
      produces 'application/json'

      response '200', 'successful' do
        schema type: :array,
          items: {
            type: :object,
            properties: {
              id: { type: :integer },
              name: { type: :string },
              email: { type: :string }
            },
            required: ['id', 'name', 'email']
          }

        run_test!
      end
    end

  end
end

We define the structure of the index endpoint of the users. We're defining the tags, procedures, and response of the endpoint.

We also specify the response. We're saying the response will be an array of objects with the properties id, name, email.

run_test! 
This will run the test for the endpoint

To specify our default host and base path, we add the following to the swagger_helper.rb file:
#frozen_string_literal: true

require 'rails_helper'

RSpec.configure do |config|
.
.
.
  config.openapi_root = Rails.root.join('swagger').to_s

.
.
.
  config.openapi_specs = {
    'v1/swagger.yaml' => {
      openapi: '3.0.1',
      info: {
        title: 'API V1',
        version: 'v1'
      },
      paths: {},
      servers: [
        {
          url: "http://{localHost}",
          variables: {
            localHost: {
              default: "localhost:3000"
            }
          }
        },
      ]
    }
  }
.
.
.
  config.openapi_format = :yaml
end

For production we'll replace localhost:3000 with the actual domain of the API.

Then we run the following command to generate the swagger documentation:
rails rswag:specs:swaggerize

Then we run rails s

And if we go to http://localhost:3000/api-docs we'll be able to see the docs.

There's the title, version, base path, host, paths, and tags of the API.
We'll also see the documentation for the index endpoint.
There's the title Users with a dropdown.
If we click on that we'll see a Try it out button.
We click execute and we should see a successful response.

We'll see the two users we created with the seed file.



# notes from class 3/21/2024
good example of documentation: Stripes ??
