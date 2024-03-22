# Services and PORO pattern
-used to organize code in a way that makes it easy to maintain and understand
-used to separate the concerns of our app

A service is a class or module that encapsulates a specific set of functionality that can be shared across the app.
-typically used to abstract complex business logic away from controllers and models
-makes the code mode modular, reusable, and easier to test

PORO - Plain Old Ruby Object

# setup
create a new Rails API
rails new services_poro_pattern --api

Then create a user model:
rails g model User name:string email:string

rails db:migrate

In the user.rb file:
class User < ApplicationRecord
  validates :name, :email, presence: true
end

In the routes:
resources :users

Then we'll create a controller for the users:
rails g controller Users create

In the users_controller.rb file:
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

# Services
We're going to extract the logic of creating a user into a service.
Create a folder called services, then a new file called user_service.rb

In the user_service.rb file:
module UserService
  module Base
    def self.create_user(params)
      user = User.new(params)
      if user.save
        user
      else
        user.errors
      end
    end
  end
end

Here we've defined the UserService module that has a Base module with a method called create_user.
When we call create_user it will create a user and return the user if it's valid.

In the users_controller.rb we'll add:
    user = UserService::Base.create_user(user_params)
    if user.valid?
      render json: user, status: :created
    else
      render json: user, status: :unprocessable_entity
    end

Another example for filtering users:

In the users controller, we'll define a new action.
def index
  users = UserService::Base.filter_users(params)
  render json: users, status: :ok
end

Then in the user_service.rb file we'll add:
module UserService
  module Base
    def self.create_user(params)
      user = User.new(params)
      if user.save
        user
      else
        user.errors
      end
    end

    def self.filter_users(params)
      users = User.all
      users = users.where(name: params[:name]) if params[:name].present?
      users = users.where(email: params[:email]) if params[:email].present?
      users
    end
  end
end

Then we'll seed some data into our users table:
User.create(name: 'John Doe', email: 'johndoe123@gmail.com');
User.create(name: 'John Doe', email: 'janedoe123@gmail.com');
User.create(name: 'Jane Doe', email: 'anotherjanedoe123@gmail.com');

rails db:seed

We can test the request is Postman with the endpoint: localhost:3000/users?name=Jane Doe

# PORO
We'll user a Ruby object to determine the success or failure of creating a user.

Create a file in the services folder called service_contract.rb:
#frozen_string_literal: true

#Standardizes what a service method should always return
module ServiceContract

  def self.success(payload)
    OpenStruct.new({ success?: true, payload: payload, errors: nil })
  end

  def self.error(errors)
    OpenStruct.new({ success?: false, payload: nil, errors: errors })
  end

end

Here we're defining a module called ServiceContract that has two methods, success and error.
They'll each return an object that has a success? method that will return true or false; it will also return the payload or errors.
We use this to define a standard way or returning the success or failure of a service.

In the user_service.rb file:
module UserService
  module Base
    def self.create_user(params)
      user = User.new(params)

      begin
         # are there any db/model errors?
        user.save!
      rescue ActiveRecord::RecordInvalid => exception
        # return an error instance
        return ServiceContract.error(user.errors.full_messages) unless user.valid?
      end

      ServiceContract.success(user)
    end

    def self.filter_users(params)
      users = User.all
      users = users.where(name: params[:name]) if params[:name].present?
      users = users.where(email: params[:email]) if params[:email].present?
      ServiceContract.success(users)
    end
  end
end

In our redefinition of the create_user method, we're using the ServiceContract module to return the success or failure.
For filter_users, we're also using the ServiceContract.

In the users controller file, add:
class UsersController < ApplicationController
  def create
    result = UserService::Base.create_user(user_params)
    if result.success?
      render json: result.payload, status: :created
    else
      render json: result.errors, status: :unprocessable_entity
    end
  end

  def index
    result = UserService::Base.filter_users(params)
    render json: result.payload, status: :ok
  end

  private

  def user_params
    params.permit(:name, :email)
  end
end

Then the controller is only responsible for handling requests and responses.
The logic of creating and filtering users is now in the services. 


# notes from class 3/21/2024
As a general rule, the actions in a controller shouldn't be super long - lots of lines of code.

German went through the example of extracting logic for creating a user and putting it in a user service.
Then the other example was a pagination method extracted out of the blogs_controller and putting it in a blog_service.
The self.paginate(params) method will return ServiceContract.success(blogs)

In the controller:
def index
    result = BlogService.paginate(params)
...

In the application controller file:

def render_error(errors:, status: :internal_server_error)
    render json: {
        success: false,
        errors: errors,
        status: status
    }, ...
end

And another method for render_success.
Then we can include those methods in the actions in the contollers.
payload = UserBlueprint.render(result.payload, view: :normal)
render_success(payload: payload)
else
render_error(result.errors)
