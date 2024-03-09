Rate limiting - 
A way to control the amount of incoming and outgoing traffic to or from a network.
Used to prevent abuse and make sure the network runs smoothly.

Create a new rails api:
rails new rate_limiting_rails_api --api

In the directory:
rails g model User name:string email:string

rails db:migrate

Create a controller:
rails g controller Users

In the UsersController:
class UsersController < ApplicationController
  def create
    user = User.new(user_params)
    if user.save
      render json: user, status: :created
    else
      render json: user.errors, status: :unprocessable_entity
    end
  end

  private

  def user_params
    params.permit(:name, :email)
  end
end

In the config/routes.rb file:
Rails.application.routes.draw do
  resources :users
end


Adding Rate Limiting
We'll use the rack-attack gem - it provides middleware to protect our app

In the Gemfile:
gem 'rack-attack'

run bundle install

In the config/application.rb file we add:
config.middleware.use Rack::Attack

Then we'll create a file config/initializers/rack-attack.rb

Here's an example of how to configure the rate limiting rules:
class Rack::Attack
  Rack::Attack.cache.store = ActiveSupport::Cache::MemoryStore.new 

  throttle('create_user/ip', limit: 1, period: 20.seconds) do |req|
    if req.path == '/users' && req.post?
      req.ip
    end
  end
end

throttle - defines the rate limiting rule
create_user/ip - the name of the rule (used to identify the rule in the logs)
limit, period - rate limiting parameters
limit is the maximum number of requests that can be made in the period
[in this example, we can only make 1 request every 20 seconds]

The do |req| ... block - describes the rule. It takes a request object.
In this example, the rule is only applied to requests to the /users endpoint that are POST requests. 
The req.ip is used to identify the client making the request.

Rack::Attack.cache.store - added to the Rack::Attack class to store the rate limiting data.
In this example we're using the ActiveSupport::Cache::MemoryStore to store the data in memory.
We can use other cache stores like Redis or Memcached (ideal for production environments)

If we try to make more than one request within 20 seconds, we should get a 429 Too Many Requests response.


Global Rate Limiting
We can apply a global rate limiter to all our requests.

An example:
class Rack::Attack
  Rack::Attack.cache.store = ActiveSupport::Cache::MemoryStore.new 

  throttle('req/ip', limit: 5, period: 5.seconds) do |req|
    req.ip
  end

    throttle('create_user/ip', limit: 1, period: 20.seconds) do |req|
    if req.path == '/users' && req.post?
      req.ip
    end
  end
end

Understand usage patterns
-Before setting rate limits, it's important to analyze an app's traffic.
Look for average request rates, peak periods, and behavior of typical and power users.
-Account for growth - it's important to accomodate current usage but also leave room for growth. 

Differentiate between user types
-Role-based limiting: implementing different rate limits for different types of users (unauthenticated users, regular users, premium users)
-IP vs User account: applying rate limits based on the user account for authenticated requests vs IP-based limiting for unauthenticated requests.

Communicate with Users
-API documentation: clearly set expectations for developers and users
-Include info about rate limits in HTTP headers of the responses (X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset)

https://github.com/rack/rack-attack

https://www.scaler.com/topics/ruby-on-rails/rate-limiting-rails/

https://www.rubydoc.info/gems/rack-attack/5.0.1

