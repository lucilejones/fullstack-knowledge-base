# Authentication vs Authorization

authentication - confirms a user is who they say they are
authorization - gives a user permission to access a resource

Authentication
ex. when a user logs in to an app with a username and password; the app verifies their identity based on the provided credentials (this is the most common way to authenticate a user).

Other ways to authenticate a user:
2FA - requires a second form of verification
This could be a code sent via SMS, email, or generated through an authentication app
Adds an extra layer of security

OAuth - allows users to login using credentials from a third-party service (Google, Facebook, Twitter)
Lets users grant the third party website or app access to their information on another website without sharing their access credentials.
It is convenient and often is secure if it leverages the robust security of major services.

token - after initial login, the server generates a token the client can use for subsequent requests.
The token is included in the headers of HTTP requests and is validated by the server.
Common in APIs and single-page apps

SSO - allows users to log in once and gain access to multiple systems without needing to re-authenticate.
Useful in coporate environments.

Passwordless authentication - allows users to log in without a password.
Often used with email, SMS, or one-time code.

Authorization
Determining whether a user has the right to access a specific resource or perform a particular action.
After logging in, the app checks if the authenticated user has the necessary permission to access a requested page or feature.

Role based access - users are assigned roles and each role has predefined permissions.

OAuth - the user's identity is authenticated and a token is given to access specific user data

Best practices for authentication:
-use strong, hashed password; BCrypt is a good option
-offer and encourage users to enable 2FA (2 factor authentication)
-temporarily lock accounts after a certain number of failed login attempts
-always use HTTPS, not HTTP, to encrypt data trasmitted between client and server
-store session ids and other sensitive info on the server (not the client side)
-keep software, libraries, and frameworks updated
-enforce minimum password length and complexity
-implement secure methods for password recovery; ensure questions are not easily guessable

Best practices for authorization:
-grant users only the permissions they need to perform their tasks; avoid giving excessive privileges
-use a role-based system and assign roles to users.
-periodically review and audit user roles and permissions and make adjustments
-where applicable, add context (ex. IP address, device type) or time-based restrictions to access


# Authentication in Rails
Rails comes with a built-in authentication system: has_secure_password
-provides methods to set and authenticate against a BCrypt password
-also adds validations to the model

where XXX is the attribute name of your desired password.

The following validations are added automatically:

Password must be present on creation

Password length should be less than or equal to 72 bytes

Confirmation of password (using a XXX_confirmation attribute)

If confirmation validation is not needed, simply leave out the value for XXX_confirmation (i.e. don’t provide a form field for it). When this attribute has a nil value, the validation will not be triggered.

Authenticating against a BCrypt password refers to verifying a user's login attempt by comparing the password they provide with the hashed password stored in the database.

Create a new project, add debug, rspec-rails, factory_bot_rails to the Gemfile.
Then run 
rails generate rspec:install

And we add the following to our rails_helper.rb:
...
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
...
end

At the top of the rails_helper.rb file we also need:
require 'spec_helper'
require 'faker'

rails g rspec:model user 
This creates a spec file for our user model.

We update the spec/factories/users.rb file:
FactoryBot.define do
  factory :user do
    username { Faker::Internet.username }
    password { 'password' }
    password_confirmation { 'password' }
  end
end

We'll create a user with username and password, and add a password_confirmation to make sure the password and password confirmation match.
We'll use BCrypt to hash the password that will be stored in the database as password_digest

In the User model we'll have has_secure_password which adds methods to set and authenticate against a BCrypt password. This method expects a password_digest attribute in the database and virtual attributes password and password_confirmation for the model.

In the spec/models/user_spec.rb file:
require 'rails_helper'

RSpec.describe User, type: :model do
  it 'is valid with a username and password' do
    user = build(:user)
    expect(user).to be_valid
  end

  it 'is not valid without a username' do
    user = build(:user, username: nil)
    expect(user).not_to be_valid
  end

  it 'hashes the password' do
    user = create(:user, password: 'password')
    expect(user.password_digest).not_to eq 'password'
  end
end

Then we create a spec file for our users controller:
rails g rspec:request users

Then in the spec/requests/users_spec.rb file:
require 'rails_helper'

RSpec.describe UsersController, type: :controller do
  describe 'POST #create' do
    context 'with valid attributes' do
      it 'creates a new user and returns a success response' do
        post :create, params: { user: attributes_for(:user) }
        expect(response).to have_http_status(:created)
        expect(User.count).to eq(1)
      end
    end

    context 'with invalid attributes' do
      it 'does not create a new user and returns an error response' do
        post :create, params: { user: attributes_for(:user, username: nil) }
        expect(response).to have_http_status(:unprocessable_entity)
        expect(User.count).to eq(0)
      end
    end
  end
end

Then we'll add a spec file for our sessions controller:
rails g respec:request sessions

In the spec/requests/sessions_spec.rb file:
require 'rails_helper'

RSpec.describe SessionsController, type: :controller do
  describe 'POST #create' do
    let!(:user) { create(:user) }

    context 'with valid credentials' do
      it 'authenticates the user and returns a success response' do
        post :create, params: { username: user.username, password: 'password' }
        expect(response).to have_http_status(:success)
        expect(JSON.parse(response.body)).to include('token')
      end
    end

    context 'with invalid credentials' do
      it 'does not authenticate the user and returns an error response' do
        post :create, params: { username: user.username, password: 'wrong_password' }
        expect(response).to have_http_status(:unauthorized)
      end
    end
  end
end

Creating a session is the process of logging in a user. We make sure a user can log in with valid credentials, but not login with invalid credentials. If the use logs in we should return a token.

Then if we run our tests, we should expect them to fail:
bundle exec rspec

We need to create our models and controllers.
rails generate model User username:string password_digest:string

Enter n to not overwrite the spec file.

rails db:migrate

Then we add has_secure_password to the User model.
class User < ApplicationRecord
  has_secure_password
end

Then if we run the test file we get 3 examples with 3 failures.
bundle exec rspec spec/models/user_spec.rb

We need to add validates :username, presence: true to the user.rb file

We also need to add bcrypt to our Gemfile

gem 'bcrypt'
bundle install

Then try the test again:
bundle exec rspec spec/models/user_spec.rb
We get 3 examples, 0 failures

Next, we'll create our user controller:
rails g controller users

In the app/controllers/users_controller.rb file:
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
    params.permit(:username, :password, :password_confirmation)
  end
end

When we use User.new or User.create with has_secure_password (in the User model), Rails automatically checks to see whether the password and password_confirmation fields match when creating a new user.

Then in our routes file we add:
Rails.application.routes.draw do
  resources :users, only: [:create]
end

Then we run our test:
bundle exec rspec spec/requests/users_spec.rb
[the notes say at this point we should get 2 examples, 0 failures, but mine gets a fail for the first test]

Create a sessions controller:
rails g controller sessions create

In the app/controllers/sessions_controller.rb file:
class SessionsController < ApplicationController
  def create
    user = User.find_by(username: params[:username])
    if user&.authenticate(params[:password])
      render json: { token: '123' }
    else
      render json: { error: 'Invalid username or password' }, status: :unauthorized
    end
  end
end

user&
The & is called the safe navigation operator. It's a Ruby operator that allows us to call a method on an object without worrying if it is nil. If the object is nil, the method will return nil instead of raising and exception.
Here, if the user is found and the password is correct, we'll return a token. Otherwise, we'll return an error message.

In our routes file we add:
post '/login', to: 'sessions#create'

Then we run our test file:
bundle exec rspec spec/requests/sessions_spec.rb



# Authorization in Rails
# Creating a JSON Web Token using the JWT gem
JWT (JSON Web Token) - a compact, URL-safe way of representing claims to be transferred between two parties. Claims are encoded as a JSON object that's used as the payload of a JSON web signmature (JWS) structure or as the plaintext of a JSON web encryption (JWE) structure. This lets the claims be digitally signed or integrity protected with a message authentication code (MAC) and/or encrypted.

A JWT is made up of three parts, separated by dots:
-header: typically consists of two parts: the type of token (which is JWT), and the signing algorithm being used (like HMAC SHA256 or RSA)
{
  "alg": "HS256",
  "typ": "JWT"
}

-payload: contains the claims; claims are statements about an entity (typically, the user) and additonal data. There are three types of claims: registered, public, private
{
  "sub": "1234567890",
  "name": "John Doe",
}


-signature: used to verify that the sender of the JWT is who they say they are, and to ensure the message wasn't changed along the way. To create the signature part we take the ecoded header, the encoded payload, a secret, the algorithm specified, and sign that.

The resulting JWT looks like: xxxxx.yyyyy.zzzzz
xxxxx is the Base64Url encoded header.
yyyyy is the Base64Url encoded payload.
zzzzz is the signature.

Once a user is logged in, each sudsequent request will include the JWT - allowing the user access to routes, services, and resources that are permitted with that token.

We'll be using the jwt gem

Logging in a user and returning a JWT:
We need to add gem 'jwt' to our Gemfile and the run bundle install

In our sessions_controller.rb file, let's update the code to use the JWT gem to create a token:
...
  private 
  
  def jwt_encode(payload, exp = 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, Rails.application.secrets.secret_key_base)
  end

This defines a private method that will encode our payload.

def jwt_encode - we define a method that includes two parameters: payload and exp; the payload represents the data we want to include in out token (typically used for storing the user's id), and the exp is the expiration time of the token. We'll set it to 24 hours from now.

payload[:exp] = exp.to_i - then we convert the expiration time to an integer. 

JWT.encode - here we encode the payload using the JWT gem. We use the secret key base (which is unique to our application).

The Rails object:
-an instance of the Rails::Application class
-created when the application boots and is available for our app to use
-also available in the console
-the container for our app's configuration, and the instance methods on the Rails object are the primary ways to configure our app

With access to the Rails object, we have access to sensitive info about our app, such as the secret key base.
The secret key base is unique to our app, is used for security purposes, and is used to encrypt sensitive data such as the Rails credentials feature (which is used to hold sensitive info like API keys, passwords, etc); we also use the secret key base to encode our payload.






# Serialization in Rails using blueprinter
