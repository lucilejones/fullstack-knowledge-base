# Introduction to Deployment
-deploying our Ruby on Rails API to a production environment


When an API is deployed, it's hosted on a server. The app becomes available to its intended users via the internet.


Environments
We need to separate our development, testing, and production environments.
When we run our app locally, we're running it in the dev environment, ex. rails s
rails c runs the console in the dev environment

We use RSpec to run our tests.

In Production, when the app is available to the public, we want to make sure the app is secure, scalable, and reliable.

Other environments:
-staging: similar to production; used to test our app before deploying to production
-UAT: the User Acceptance Testing environment; used to test our app with real users before deploying (can be internal or external users)
-QA: the Quality Assurance environment; used to test our app for quality standards before deploying

Example workflow: work in development, test in test, test a live version in staging, then deploy to production when ready.


Deploying an API vs deploying a frontend app
Deploying the API is deploying the backend. It will require a series of executions to run properly, and it will also require a database.
When a user requests the domain of our API, the server will send the data the user requested. This could be in the form of JSON, XML, or another format. The user's browser will then use that data to updated the UI.

Deploying the frontend is deploying the user interface. Deploying the HTML, CSS, and JS is much simplier than deploying a backend.
When a user requests the domain of our frontend app, the server sends the HTML, CSS, and JS files to the user's browser. For frameworks like Angular, React, or Vue, the server sends the index.html file, and that file will then load the JS and CSS files that make up the UI.


# Deploying to Render
Options: Heroku, AWS, Google Cloud, Railway, Render, Hatchbox, and others.

Render offers a free tier and is simple to use.


Setting up an example Rails API for deployment
Create a new rails application:
rails new render_deployment_example --api

Then we'll generate a model:
rails g model User username:string

And a users controller:
rails g controller Users index create

In the config/routes.rb file:
Rails.application.routes.draw do
  resources :users, only: [:index, :create]
end

In the users_controller.rb file:
class UsersController < ApplicationController
  def index
    render json: User.all, status: :ok
  end

  def create
    user = User.new(username: params[:username])

    if user.save
      render json: user, status: :created
    else
      render json: user.errors, status: :unprocessable_entity
    end
  end
end


Configuring our Rails API for deployment
We'll need to push this to github and make sure to commit.

Most services (including Render) require postgresql as the database, so we need to update that in our Gemfile:
group :production do
  gem 'pg'
end

group :development, :test do
  gem "sqlite3", "~> 1.4"
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mswin mswin64 mingw x64_mingw ]
end

We need to remove sqlite3 from all environments and add it to dev and test only.
We'll add pg to the production environment only.

bundle install
[I get an error here saying that it encountered an error:
An error occurred while installing pg (1.5.5), and Bundler cannot continue.]
I ran the command:
sudo apt install postgresql-contrib libpq-dev

Then did bundle install again and it worked.


We'll change the database configuration in the config/database.yml file:
production:
    adapter: postgresql
    encoding: unicode
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
    url: <%= ENV['DATABASE_URL'] %>

In the bin directory we'll create a file called render-build.sh and add the following:
#!/usr/bin/env bash
# exit on error
set -o errexit
bundle install
bundle exec rake db:migrate

This file will be used to build our app on Render. It will install our gems and run our migrations.

#!/usr/bin/env bash
Called a shebang; tells the operating system what interpreter to user to run the file. In this case, we're using bash.

set -o errexit
This line tells the operating system to exit the script if any command returns a non-zero status. It will prevent the script from continuing if a command fials.

In the root of the project, we create a file called render.yaml:
databases:
    - name: renderdeploymentexample
      databaseName: renderdeploymentexample
      user: renderdeploymentexample
      plan: free

services:
    - type: web
      name: renderdeploymentexample
      runtime: ruby
      plan: free
      buildCommand: './bin/render-build.sh'
      # preDeployCommand: "./bin/rails db:migrate" # preDeployCommand only available on paid instance types
      startCommand: './bin/rails server'
      envVars:
          - key: DATABASE_URL
            fromDatabase:
                name: renderdeploymentexample
                property: connectionString
          - key: RAILS_MASTER_KEY
            sync: false
          - key: WEB_CONCURRENCY
            value: 2 # sensible default

This file will be used to configure our app on Render. It will create a database and web service falled renderdeplomentexample - we'll need to change the name to match the name of our app:
renderdeploymentexamplelucilejones

When we deploy our app, we'll run the buildCommand to install our gems and run our migrations. We'll also run the startCommand to start our server.

Then envVars section is used to set environment variables for the app.
DATABASE_URL: this environment variable is used to connect to our database; it will be set to the connection string of our database.
RAILS_MASTER_KEY: this environment variable is used to decrypt our credentials; it will be set to the master key of our app [we'll learn more about this]
WEB_CONCURRENCY: this environment variable is used to set the number of worker processes that will be used to run our app; it will be set to 2. [we'll explore this topic more later]

Whenever deploying an app, we need to change the name of the database and webservice to match the name of our application.

Commit and push to Github


Deploying with Render
We'll sign up for an account with Render and use our GitHub account.
https://render.com/

Click on the navlink that says "Blueprints"
Click on the button that says new blueprint instance
Search for the repository you just created and click connect.
[here I had to click on Configure account, install Render on my GitHub account, enter my password, and select the repository I wanted to connect to Render]
It will ask for a blueprint name. You can name it whatever you want. I will name mine Default Rails Render
Enter the Rails Master Key

The Rails Master Key is used to decrypt our credentials specific to the environment we're in. It's different for every app and environment.
We'll set this to the master key of our app.
In the config/master.key file
We copy the key and then add it to the Rails Master Key field.

Then we click Apply - this may take some time.

Once that's finished, we go to the Dashboard and click on the link to the one that shows the type to be Web Service.


If we get an error it might mean that the platform is not supported. We can run the following:
bundle lock --add-platform x86_64-linux

Then we can commit and push to Github.

We'll see the URL to our app. If we click on it, we can see the app running.
However, we do get a 404 error because our API doesn't have a root route.

We can test it out by using postman to create a user.

We can send a POST request to 
https://renderdeploymentexamplelucilejones.onrender.com/users

We can send
{
	"username": "test"
}

And get a response:
{
    "id": 1,
    "username": "test",
    "created_at": "2024-03-01T19:08:44.922Z",
    "updated_at": "2024-03-01T19:08:44.922Z"
}


Render only allows for one free service for the database and web service. We'll need to delete one service before we can deploy another one. 


# notes from class 3/4/2024

Getting a CORS error after deploying the frontend:
We need to add frontend url to the origins of the cors.rb file
(In initializers in the config folder in the backend api)
