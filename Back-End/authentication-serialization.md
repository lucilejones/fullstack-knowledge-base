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



# Authorization in Rails


# Serialization in Rails using blueprinter
