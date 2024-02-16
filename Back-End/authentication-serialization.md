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

If we run rails console and type Rails.application.secrets.secret_key_base we'll see the secret key base for our app. This is just for the development environment because that's what we have access to in the console. The secret key base is different for every environment.

Then we update our sessions controller to use the jwt_encode method:
    if user&.authenticate(params[:password])
      token = jwt_encode(user_id: user.id)
      render json: { token: token }, status: :ok

We encode the payload and store it in a variable called token. We're passing in the user's id as the payload.
In the rails console if we create a user we should then be able to login in postman with that user's username and passoword.
User.create(username: 'username', password: 'password', password_confirmation: 'password')

We'll get a response back that looks like this:
{
    "token": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJleHAiOjE3MDYzNzk3MDZ9.a0xAX98--3NIRLs6qXr-HM5D5Zyd7ropPGfmuNUeCao"
}


Encryption:
-a two-way function in which data is converted into a format that cannot easily be understood by unauthorized people. It can be decrypted back to its original form if the person has the right key.
-used when we need to securely transmit sensitive info over a network (like credit card info)

One-way hashing:
-a one-way function in which data is converted into a fixed length string of characters. It cannot be decrypted back to its original form. It's used to verify the integrity of data. (Using BCypt to hash our passwords)
-when a user creates a password, it's hashed using a hash function like BCrypt and the resulting hash is stored in the database. When the user logs in, the password they enter is hashed again and the resulting hash is compared to the one stored. Since hashing is one-way, even if someone gains access to the database, they cannot reverse the hashes back to passwords.

Encoding:
-the process of converting data from one form to antoher; it doesn't provide any security, it just represents data in a different format
-used when data needs to be transformed into a format that's compatible with different systems or protocols. For example, when embedding binary data in an XML or JSON payload we might use Base64 encoding to convert the binary data into a text representation that can be easily included in the text-based payload.


JWTs are encoded which makes them URL-safe and easy to pass in HTTP headers
-they're stateless, which means the server doesn't need to keep track of the user's session
-the signature is encrypted with a secret key

The server decodes the JWT to retrieve the header and payload.
The server then verifies the signature by comparing it against its own computer signature using the secret key.
If the signature is valid, the server trusts the claims within the JWT and authenticates the user.


Authenticating a user using a JWT

We're going to make certain routes protected so only authenticated users can access them.
Let's add the :show method to out config/routes.rb file

Then in the users controller we add:
def show 
    user = User.find_by(id: params[:id])

    render json: user, status: :ok
end

Since all controllers will inherit from the ApplicationController, we can include logic there to be used to authenticate the user.

app/controllers/application_controller.rb:
class ApplicationController < ActionController::API
  def authenticate_request
    header = request.headers['Authorization']
    header = header.split(' ').last if header
  end
end

We define a method called authenticate_request.
To decode the token we need to access the Authorization header.
The most common type of authentication is the Bearer authentication scheme which uses a security token called the Bearer token. 
We use split to split the header into an array (splitting by the space character) and then grabbing the last element of the array which is the token.

We then wrap the process of decoding the token in a begin/rescue block:
    begin
      decoded = JWT.decode(header, Rails.application.secrets.secret_key_base).first
      @current_user = User.find(decoded['user_id'])
    rescue JWT::ExpiredSignature
      render json: { error: 'Token has expired' }, status: :unauthorized
    rescue JWT::DecodeError
      render json: { errors: 'Unauthorized' }, status: :unauthorized
    end

If the token has expired, we'll return an error message, if the token is invalid we'll return an error message. Otherwise, we'll find the suer and store it in an instance variable called @current_user.

When we call JWT.decode we include the header which has the token and the secret key base. We use the secret key base to verify the signature of the token.

Then we can use the authenticate_request method to protect our routes. In the users controller file we'll add:
before_action :authenticate_request, except: [:create]

We want to call this before the show action but not the create action (so we put that in except). We want users to be able to create an account without being authorized.

In postman, to send a request with a token we go to the Authorization tab, then for type we select Bearer Token. Then we paste in the token we got back from the login request.


One concern with using JWT for session management is the difficulty in invalidating a token once it's been issued. JWTs remain valid until they expired. This can be problematic when immediate revocation is necessary, like for a user logout or account suspension.
JWTs are also criticized for being larger than traditional session tokens, which can incread the load on network traffic because they're included in every HTTP request.


Token Storage and Revocation
Creating a customized token (not a JWT) is an alternate way to authenticate a user. 


# Serialization
The process of converting data structures of objects into a format that can be stored or transmitted and reconstructed later. 
examples:
-when we send data from the client to the server, it needs to be serialized into a format that can be transmitted over the network
-when we store data in a database, it needs to be serialized into a format that can be stored

advantages:
-reduces the amount of data sent over the network, which improves performane and reduces bandwith usage
-allows us to control what data is sent to the client (we can exclude sensitive info like passwords or credit card numbers)
-allows us to control how the data is formatted (like dates)
-allows us to include additional info that's not part of the data model (like links to related resources)

Currently when we send a request to localhost:3000/users/1 we get the following response:
{
    "id": 1,
    "username": "username",
    "password_digest": "$2a$12$rRGqSRKChBUMRs1eIvVjDOeXgZXy17NUViGizydc/EhNGc2.pL.Bu",
    "created_at": "2022-01-25T01:11:32.201Z",
    "updated_at": "2022-01-25T01:11:32.201Z"
}

But we don't want to send the password_digest to the client, and we probably don't need to send the created_at and updated_at attributes.


We'll be using the gem blueprinter


# Serialization in Rails using blueprinter
To the Gemfile add gem 'blueprinter'
Then run bundle install

To create a serializer for the user model:
rails g blueprinter:blueprint user

This creates a new file:
app/blueprints/user_blueprint.rb:
# frozen_string_literal: true

class UserBlueprint < Blueprinter::Base
  identifier :id

  fields :username
end

Here we define the identifier (id) and the fields that we want to serialize (in this case username).
If we serialize a user record using this blueprint, we'll get:
{
  "id": 1,
  "username": "username"
}

We're not serializing the password_digest, created_at, and updated_at attributes.

Then in the users controller (in the show method), we update:
from
render json: user, status: :ok
to
render json: UserBlueprint.render(user), status: :ok

We can create different views for the blueprinter for different use cases of what we want to serialize. For example, sometimes we might want to serialize the user's email address but not other times. 

In our user blueprint we'll update it to have two different views (one with just the username serialized, and one that also includes the created_at and the updated_at):
# frozen_string_literal: true

class UserBlueprint < Blueprinter::Base
  identifier :id

  view :normal do
    fields :username
  end

  view :extended do
    fields :username, :created_at, :updated_at
  end
end

Then in the users contoller, we specify the view for the blueprint to render:
render json: UserBlueprint.render(user, view: :normal), status: :ok


We can also serialize associations.
We'll create a new model Post.
rails g model Post title:string body:text user:references

Then run rails db:migrate

The Post model will have belongs_to :user
And we add has_many :posts to the User model

To create a serializer for the post model:
rails g blueprinter:blueprint post

In the app/blueprints/post_blueprint.rb file:
class PostBlueprint < Blueprinter::Base
  identifier :id

  view :normal do
    fields :title, :body
  end
end

We've defined a view called noraml that will serialize the title and body attributes.
Then we can include the posts association in our user blueprint:
...identifier :id

association :posts, blueprint: PostBlueprint, view: :normal
...

We're using the method call association to serialize the posts association; we're passing in the method name - posts - indicating the user model has a posts association.
We're also passing in the blueprint (the post blueprint and the view).

Then we can use the rails console to save some data in the database.
[added three posts to user 1]
Then when we sent a GET request to localhost:3000/users/1
We'll get back an object with the user id, username, and an array of posts.


This might not be practical if a user has hundreds of posts, so we can change our code to only get the first post from the user.
  association :posts, blueprint: PostBlueprint, view: :normal do |user|
    user.posts.first
  end



# Notes from class 2/12/2024

it 'hashes the password using BCrypt' do
  user = create(:user, password: 'password')

  expect(user.password_digest).to_not eq 'password'

  expect(BCrypt::Password.new(user.password_digest)).to be_truthy

  expect(Brypt::Password.new(user.password_digest).is_password?('password')).to be true
end

password_digest is how the database stores the password

We add gem 'bcrypt' to the Gemfile, run bundle install

add has_secure_password to the User model

rails g migration AddPasswordDigestToUsers password_digest:string

Then rails db:migrate

Then in the spec/factories/users.rb we add:

password { 'password' }
password_confirmation { 'password' }

These get added to a user when we have the has_secure_password attribute
We can put those properties on a user, but they don't get stored in the database as password and password_confirmation, they get stored just as password_digest

Then in the routes:
resources :users, only: [:create]

We generate a users controller:

def create
  user = User.new(user.params)
  if user.save
    render json: user, status: :created
  else
    render json: user.errors, status: :unprocessable entity
  end
end

private

def user_params
  params.permit(:username, :email, :first_name, :last_name, :password, :password_confirmation)
end

rails routes --expanded
(Can list our available routes in the console)


Allowing users to login
JSON Web Tokens

We use a new controller called a sessions controller

we add gem 'jwt' to the Gemfile and run bundle install

To the routes file we add:
post '/login', to: 'sessions#create'

rails g controller Sessions create

In the sessions_controller.rb file:
def create
  user = User.find_by(username: params[:username])

  if user.authenticate(params[:password])
    token = jwt_encode(user_id: user.id)
    render json: { token: token }, status: :ok
  else
    render json: { error: "Unauthorized" }, status: :unauthorized
  end
end

private

def jwt_encode(payload, exp = 24.hours.from_now)
  payload[:exp] = exp.to_i
  JWT.encode(payload, Rails.application.secrets.secret_key_base)
end

Rails keeps secrete keys, API keys, etc very secure.


Then in the applications_controller.rb we write a method to say which we endpoints we want a user to be authenticated in order to use:

def authenticate_request
  header = request.headers['Authorization']
  header = header.split(' ').last if header

  begin
    decoded = JWT.decode(header, Rails.application.secrets.secret_key_base).first
    @current_user = User.find(decoded['user_id'])
  rescue JWT::ExpiredSignature
    render json: { error: 'token expired' }, status: :unauthorized
  rescue JWT::DecodeError
    render json: { error: "Unauthorized" }, status: :unauthorized
  end
end

Then we user the authenticated_request method before other methods where we want a user to be authenticated.

In the BlogsController file:

before_action :authenticate_request

Now a user will need to be authenticated in order to use any of the blogs methods.

In postman, we'll need to create a new user with a POST request. (With first_name, last_name, email, username, password, password_confirmation.)

Then we do a POST request to /login with the username and password.
We'll get a token back. (A long string.)
Then for blogs requests we'll need to add the token to the headers.
We click on Authorization, and use the Bearer Token. We paste the token into the Token field.