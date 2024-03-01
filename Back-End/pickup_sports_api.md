# Pickup sports api ERD
https://www.youtube.com/watch?v=rkOdoZ8nCho&list=PLj9uZuEI0pczp_qgNf6oAGvwBTLORcT4u&index=1

- a user has one profile (one table for user, one table for profile)
one-to-one relationship

- a user can have many posts (the post table has a foreign key for the user)
one-to-many (optional)

- a post can have many comments and a comment belongs to a post
- an event can have many comments and a comment belongs to an event

- the comment table has CommentableID and CommentableType
polymorphism
the table can be in relationships with multiple tables
(a post can have many comments, an event can have many comments, etc)
(a separate table for post comments and event comments gets redundant; unless one table uses several columns that the other doesn't)

- the event has a foreign key for the user id, the creator of the event

- a user can have many events, and an event belongs to a user

- an event can have many event participants through users, and a user can have many event participants through events. 
We create a join table Event_Participant: users can see how many events they're participating in, and the events can see how many users are participating in each. 
It needs two foreign keys, the userid and the eventid.
We'll have a third column rating, so we need the event_participant to have an unique id (primary key).

- a location table relates to the user and the event
polymorphism

- a sport can have many events and an event can have many sports
many-to-many 
the event_sport table has two foreign keys (sportid and eventid), but no primary key

# creating the users table - migrations, rails console, model
rails new pickup_sports --api

We use the gem "sqlite3" for development (for the database) because it's lightweight, but we should use PostgreSQL for production.

"puma" starts up the server
"debug" sets up for testing

First we'll create the User entity.
rails generate model User username:string email:string first_name:string last_name:string
(We're ignoring the password for now.)

This creates a migration file, a model file, a test unit, and other test files.

The migration file has a timestamp and a title names for what we're doing, creating users.
db/migrate/20240111021324_create_users.rb
It creates a class CreateUsers
It will change the db into creating the users table; we'll have four columns plus timestamps.

We should also include databse validations and add these to the migration file.
Then we can run rails db:migrate

It creates the users table and we get a schema.rb file.

If we want to change the latest migration file, we can run rails db:rollback and that will remove those latest changes from the database.
Then we can update the migration file and run rails db:migrate again.

In the app/models/user.rb file, we set up the User class (which inherits from ApplicationRecord):


We can run rails console to be able to interact with the database.
We can create a User. 
User.create(username: "janedoe123", email: "janedoe123@email.com", first_name: "Jane", last_name: "Doe")

# adding built-in validations to the user model
database validations, model validations, and frontend validations

In the user.rb:
We use the method validates, with built-in validators and custom validators.
We can have custom errors from the model but the database won't do that.

If we hvea the rails console open and then make adjustments to the model, etc, we'll need to reload the terminal:
reload!

By default Rails creates an id for each instance. If the id is ever nil that means the record wasn't created correctly.

We can enter (in the rails console) user.errors or user.errors.messages or user.error.full_messages
This will print out error information

We can add a uniqueness validator: uniqueness: true
and a length validator: length: {minumun: 3, maximum: 30}
Another important validator for the email is format: {with: URI::MailTo::EMAIL_REGEXP}
This is a built-in validator with Rails. This will make sure the value compares to a standard email. (We need the key with)

Validators allow us to prevent saving data in the database that isn't consistent or is difficult to work with (example, a username 1,000 characters long).

# adding custom validators to the user model
We use the method validate and then the name of the validator.
validate :validate_username
We'll create a private method.
private
def validate_username
    unless username =~ /\A[a-zA-Z0-9_]+\Z/
        errors.add(:username, "can only contain letters, numbers, and underscores, and must include at least one letter or number")
    end
end

We're saying the username needs to match or be similar to this Regex expression. (Lots of examples that we can find by googling.) This particular one is saying from the beginning of the username til the end, it can have lowercase letters through the alphabet, uppercase letters through the alphabet, numbers 0-9 and underscores.

errors is a pre-built property that exists on the instance itself. 
We user errors.add followed by the column that has errors, in this case the username, as the first argument, and then the message.

# one to many relationship
has_many association, belongs_to associate, User and Posts
A user can have many posts and a post belongs to a user.

rails g model Post user:references content:text

the user:references creates a foreign key that points to the user 
text can hold more characters than a string type

Then we run rails db:migrate
This will update our schema file.

The post.rb model file will include
belongs_to :user

This creates a method user on top of a Post instance that will find the user by the user_id.

We want to also include validations
validates :content, presence: true, length: {maximun: 2000}

Then we can create a post:
Post.create(content: "This is a post", user_id: 1)

Then we can use Post.find(1).user and it will give back the user information for that user connected to that post.

If we want to get the user's posts, we have to add the association to the user model:
has_many :posts

Then we can do User.find(1).posts 

Post.new vs Post.create
create will automatically try to save to the database, while new doesn't save it in the database yet. We'd then need to do post.save (if we'd saved it in a variable post).

Or we can do user.posts << post
(if we didn't already put a user_id when we set up the new post)

We're referencing the User saved in the user variable and (using the shovel operator) pushing the new post into the array of that user's posts.

# one to one relationship
has_one association, belongs_to association - User and profile

A user can have just one profile and a profile belongs to a user.

rails g model Profile user:references bio:text

The Profile model will have belongs_to :user
(this will be set up when the model file gets created)

We run rails db:migrate

We need to add has_one :profile to the User model

# polymorphic
When a record in a table can belong to more than one record in other tables.

The Comment table gets a UniqueId (prmiary key), UserID (foreign key), Content, CommentableID (which will come from the post or event it's associated with), and CommentableType (this will let us know if it's associated with the post or event).

rails g model Comment user:references commentable:references{polymorphic} content:text
(This will create the migration file)

rails db:migrate
(When we run this migration it updates the schema with the comments table; it will create the columns for the commentableID and the commentableType)

Then in the User model:
mas_many :comments, dependent: :destroy
(A choice we can make: when the user is deleted, these will be deleted)
(We can do this with the posts and profile also, but need to be careful with this code.)

When Rails set up the Comment model, it comes with the following code:
class Comment < ApplicationRecord
  belongs_to :user
  belongs_to :commentable, polymorphic: true
end

Then we need to set up the Post model (and Event model) correctly.
has_many :comments, as: :commentable

Then we run rails console

user = User.first
user.comments (at first will return an empty array)

post = Post.first

user.comments.create(commentable: post, content: "This is a comment")
[user.comments is an array, but it also comes with methods, like .create()]


We'll do the same process for Location - this can be connected to User or Event.

rails g model Location locationable:references{polymorphic} zip_code:string city:string state:string country:string address:string

Then rails db:migrate

Then in the user.rb file we set
has_one :location, as: :locationable

[we could also add come validations to the Location model]

rails console
user = User.first
Location.create(locationable: user)

user.location

# many to many relationship
has_many :through association - Evants and Users

rails g model Event user:references content:text start_date_time:datetime end_date_time:datetime guests:integer

Then rails db:migrate

In the Event model we'll add:
validates :start_date_time, :end_date_time, :guests, presence: true

has_one :location, as: :locationable, dependent: :destroy
has_many :comments, as: :commentable, depended: :destroy

Then we need to make the association with the User. (In the model)

has_many :events

Then in the rails console:
user = User.first
user.events.create(guests: 5, start_date_time: DateTime.now, end_date_time: DateTime.now + 1.day)

event = Event.first
Then we can check event.comments and event.location

In order for the event to have many participants (users), the Event_Participant table will keep track of which users are particpating in which event and which event has which participants.

rails g model EventParticipant user:references event:references rating:integer

In the event.rb we include:
has_many :event_participants
has_many :users, through: :event_participants

Then in the user.rb file:
has_many :event_participants
has_many :events, through: :event_participants

Then we can grab all the EventParticipants with a specific user id which will also grab the event by that user id.

rails console

user = User.first
event = Event.create(start_date_time: DateTime.now, end_dat_time: DateTime.now + 1.day, user:user, guests: 5)

user.events will get an error
We need to set it up so we can get both the list of events the user is participating in and also separately the list of events the user has created.
has_many :created_events, class_name: 'Event', foreign_key: 'user_id'

At this point, we'd forgotten to run rails db:migrate

user.events will be all the events that user is participating in (accessed through the event_participants)
user.created_events will be all the events that user is associated with directly (that they created)

EventParticipant.create(user: user, event: Event.first)
Event.first.users

# many to many relationships
has_and_belongs_to_many association - Events and Sports

A sport can have many events and an event can have many sports.
We'll use EventSport as a join table.
We might want an even to have many sports, or to filter events by sports.

Previously, we created a third model EventParticipant, but we won't create a separate model for the EventSport; we don't need it.

rails g model Sport name:string
rails g migration CreateJoinTableEventSport event sport

We need to uncomment the two lines of code in the migration file and then run rails db:migrate

In the Sport model, we add:
validates :name, presence: true

has_and_belongs_to_many :events

This assumes there's a third table events_sports (which has to be in alphabetical order) which will include a sport_id as a foreign key.

In the Event model:
has_and_belongs_to_many :sports

rails console
event = Event.first
event.sports

sport = Sport.create(name: "basketball")
event.sports << sport
event.sports

# introduction to controllers, routes, resources, and postman - Index action
In the config/routes.rb file:
We define the routes we can send requests to.
Rails.application.routes.draw do
  get '/users', to: 'users#index'
end

We use the to keyword to say which specific controller is going to handle this request.
A controller is the intermediary between the database and the requestor.
The users controller is going to handle this request and the action that will be triggered is called index.

In the terminal we run:
rails g controller users index

This creates a file in app/controllers/users_controllers.rb

rails s 
Will start the server.

Then we define the index action in the UsersController
def index
    users = User.all

    render json: users, status: 200
end

By default in each action we have access to all the models. (We don't have to import anything into this file.)

# Resources - show action
To get one specific user:

In the routes.rb file:
get '/users/:id', to: 'users#show'

Then in the users_controller file:
def show
    user = User.find(params[:id])

    render json: user, status: 200
end

# Resources - create action
To create a new user:

In he routes.rb file:
post '/users', to: 'users#create'

Then in the users_controller file:
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
    params.require(:user).permit(:username, :email, :first_name, :last_name)
end

# Resources - update action
In the routes.rb file:
put 'users/:id', to: 'users#update'

In the users_controller.rb file:
def update
    user = User.find(params[:id])

    if user.update(user_params)
        render json: user, status: :ok
    else
        render json: user.errors, status: :unprocessable_entity
    end
end

Because we have repeated code in the show and update functions, we can define another private method called set user.

def set_user
    @user = User.find(params[:id])
end

We define an instance variable in which we set this to the User.find.
Then we execute that method before executing the update action or the show action.
At the very top of the controller file, we execute a method before_action (but only for the show and the update)

before_action :set_user, only: [:show, :update]

In order to make variables accessible between actions, we need to use the @

if @user.update(user_params)
    render json: @user, status: :ok
else
    render json: @user.errors, status: :unprocessable_entity
end

# Resources - destroy action
In the routes.rb file:
delete '/users/:id', to: 'users#destroy'

In the users_controllers.rb file:
(we need to add the :destroy method to the before_action method)
def destroy
    if @user.destroy
        render json: nil, status: :ok
    else
        render json: @user.errors, status: :unprocessable_entity
    end
end

We might want better error message for the delete, like why the resource couldn't be deleted.


# nested routes - getting user posts

If we put (in the routes.rb file):
resources :users

Rails will automatically define the index, show, create, and destroy functionality.
We don't have to necessarily define those routes ourselves.

We can include additional information on the routes (for example, to get the posts)

get '/users/:id/posts', to: 'users#posts_index'

Then we need to define that in the users_controller.rb file:
(We include :posts_index at the top in the before_action :set_user - since we're using the @user from the set_user method - but we take this out becuase @user doesn't end up working)
def posts_index
    user_posts = @user.posts

    render json: user_posts, status: :ok
end

We can also define the custom routes in a code block. That will allow us to write less code:
resources :users do
    get 'posts', to: 'users#posts_index'
end

However, it creates a problem because it makes the path /users/user_id/posts.
user_posts = @user.posts will no longer work
So we'll define the user in the posts_index function (instead of using @user):
def posts_index
    user = User.find(params[:user_id])
    user_posts = user.posts

    render json: user_posts, status: :ok
end

So we need to take :posts_index out of the before_action.

The to set up our routes for the posts:

resources :posts
(This will set up the routes for index, create, show, update, destroy)


# post resource - create, update, and destroy

Then we need to define the methods. (For now we'll hold off on defining the index and show methods. These are going to be a litte more complicated.)

We need to create a controller file for the posts:
rails g controller posts

Then we can copy the code from the users_controller file and paste into the posts_controller file. But we'll need to switch all the user variables to post.
(right click on user and change occurances)

We'll delete the index and show methods, and the posts_index method.

We also need to permit different params (in the post_params method):
(we also don't need to require - that's mainly for the fullstack Ruby on Rails)

def post_params
    params.permit(:content, :user_id)
end

We also need to remove :show from the before_action

In the routes.rb file we want only:
  resources :posts, only: [:create, :update, :destroy]


# testing in rails using RSpec - setup and User Model
In the Gemfile (in :development, :test):
  gem "rspec-rails"
  gem "factory_bot_rails"
  gem "faker"

Then run bundle install
Then rails generate rspec:install
This creates the spec directory; and every time we generate a model or controller is will create spec files for those also.

We can remove the minitest directory with rm -rf test

In the rails_helper.rb file we need to require 'faker'

To create a spec file for the user model:
rails g rspec:model user

In the rails_helper.rb file we also need to add (in the RSpec.configure section):
config.include FactoryBot::Syntax::Methods

We'll add some tests to the user_spec.rb file:

context 'Validation tests' do
    it 'is not valid without a first name' do
        user = build(:user, first_name: nil);
        expect(user).not_to be_valid
    end
end

build is a FactoryBot method

Then in the spec/factories/users.rb file:
username { Faker::Internet.username(specifier: 3..20, separators: %w(_)) }
email { Faker::Internet.email }
first_name { Faker::Name.first_name }
last_name { Faker::Name.last_name }

Here we use Faker to generate data

bundle exec rspec
This runs the project verions on rspec and not what we have installed globally.
This is important if we're using different versions on different projects.

We can also create tests for a missing last_name, etc. 

Then we'll write tests for uniqueness:

context 'Uniqueness tests' do
    it 'is not valid without a unique username' do
        user1 = create(:user)
        user2 = build(:user, username: user1.username)

        expect(user2).not_to be_valid
        expect(user2.errors[:username]).to include("has already been taken")
    end
end

Tests for deleting everything associated with a user:

context 'destroy user and everything dependent on it' do
    let (:user) {create(:user)}
    let (:user_id) {user.id}

    before do
        user.destroy
    end

    it 'deletes profile' do
        profile = Profile.find_by(user_id: user_id)
        expect(profile).to be_nil
    end

    it 'deletes location' do
        location = Location.find_by(locationable_id: user_id)
        expect(location).to be_nil
    end

    it 'deletes posts' do
        posts = Post.where(user_id: user_id)
        expect(posts).to be_empty
    end

    it 'deletes comments' do
        comments = Comment.where(user_id: user_id)
        expect(comments).to be_empty
    end
end


# Testing requests using RSpec - Posts Controller
rails g rspec:request Posts

This will generate the files to set up test cases. There will be a default set up:
  describe "GET /posts" do

  end

Then we can write our tests:
let(:post) {create(:post)}

#we have to actually call this to create the post
before do
    post
    get "/posts"
end

it "returns a successful response" do
    expect(response).to be_successful
end

it "returns a response with all the posts" do
    expect(response.body).to eq(Post.all.to_json)
end

Then we'll add a new describe section. (We'll do one each for show, create, update, destroy.)

describe "GET /post/:id" do
    let(:post) {create(:post)}

    before do
        get "/posts/#{post.id}"
    end

    it "returns a successful response" do
        expect(response).to be_successful
    end

    it "returns a response with the correct post" do
        expect(response.body).to eq(post.to_json)
    end
end

For the create and the update, we'll need to create the params we're sending.
We'll also create a factory for posts.

describe "POST /posts" do
    context "with valid params" do
        let (:user) {create(:user)}

        before do
            post_attributes = attributes_for(:post, user_id: user.id)
            post "/posts", params: post_attributes
        end

        it "returns a successful response" do
            expect(response).to be_successful
        end

        it "creates a new post" do
            expect(Post.count).to eq(1)
        end
    end

    context "with invalid params" do

        before do
            post_attributes = attributes_for(:post, user_id: nil)
            post "/posts", params: post_attributes
        end

        it "returns a response with errors" do
            expect(response.status).to eq(422)
        end
    end
end

describe "PUT /posts/:id" do
    context "with valid params" do
        let(:post) {create(:post)}

        before do
            post_attributes = attributes_for(:post, content: "updated content")
            put "/posts/#{post.id}", params: post_attributes
        end

        it "updates a post" do
            post.reload
            expect(post.content).to eq("updated content")
        end

        it "returns a successful response" do
            expect(response).to be_successful
        end
    end

    context "with invalid params" do
        let(:post) {create(:post)}

        before do
            post_attributes = {content: nil}
            put "/posts/#{post.id}", params: post_attributes
        end

        it "returns a response with errors" do
            exepct(response.status).to eq(422)
        end
    end
end

describe "DELETE /posts/:id" do
    let (:post) {create(:post)}

    before do
        delete "/posts/#{post.id}"
    end

    it "deletes a post" do
        expect(Post.count).to eq(0)
    end

    it "returns a successful response" do
        expect(response).to be_successful
    end
end

To create the factory for posts:
rails g rspec:model Post

Then in the spec/factories/posts.rb file:
factory :post do
    content { Faker::Lorem.paragraph }
    user
end

The user will refenerence our other factory for users.

Next we need to define out routes.
In the config/routes.rb file:
We change it to just say resources :posts

Then in the app/controllers/posts_controller.rb file:
before_action... Here we need to add :show

def index
    posts = Post.all
    render json: posts, status: :ok
end

def show
    render json: @post, status: :ok
end


# tests for users controller - another example of testing requests
rails g rspec:request Users

Then we copy everything from posts_spec.rb into users_spec.rb
(The users controller and posts controller have almost the exact same setup.)

Then everywhere it says Post, or posts we change to User or users

This will mostly work (with some tweaks - see files), but we also need to update our users_controller file.
The user_params can just be:
params.permit(:username, :email, :first_name, :last_name)

Then the tests will all pass.


# testing our events model using Rspec, factory bot, and faker
We'll create a test file for the event model:
rails g rspec:model Event

This will create two files: spec/factories/events.rb and spec/models/event_spec.rb

In the factories/events.rb file:
FactoryBot.define do
    factory :event do
        user
        content { Faker::Lorem.paragraph }
        start_date_time { Faker::Time.between(from: DateTime.now + 1, to: DateTime.now + 2) }
        end_date_time { Faker::Time.between(from: DateTime.now + 3, to: DateTime.now + 4) }
        guests { Faker::Number.between(from: 1, to: 10) }
    end
end

Then we'll write the tests in the event_spec.rb file:

require 'rails_helper'

RSpec.describe Event, type: :model do
    context "validations" do
        it 'is not valid without a user' do
            event = build(:event, user:nil)
            expect(event).not_to be_valid
        end

        it 'is not valid without a title' do
            event = build(:event, title: nil)
            expect(event).not_to be_valid
        end

        it 'is not valid with start_date_time before current time' do
            event = build(:event, start_date_time: DateTime.now - 1)
            expect(event).not_to be_valid
        end

        it 'is not valid with start_date_time after end_date_time' do
            event = build(:event, start_date_time: DateTime.now + 1, end_date_time: DateTime.now)
            expect(event).not_to be_valid
        end
    end

    context "associations" do
        it 'belongs to a user' do
            event = build(:event)
            expect(event.user).to be_present
        end

        it 'has many comments' do
            event = create(:event)
            create_list(:comment, 3, commentable: event)

            event.reload
            expect(event.comments.count).to eq(3)
        end

        it 'has many sports' do
            event = create(:event)
            create_list(:sport, 3, events: [event])

            event.reload
            expect(event.sports.count).to eq(3)
        end
    end
end


We get five failed tests.

We'll run a migration file to add a title to the events table:
rails g migration AddTitleToEvents

In the migration file:
def change
    add_column :events, :title, :string
end

Then run rails db:migrate

In the factories/events.rb file we need to add:
title { Faker::Lorem.sentence }

Then run bundle exec rspec (still fail 5 tests)

To the app/models/event.rb we'll add:
validates :title, presence: true

We'll also add a custom validator:

validate :start_date_time_cannot_be_in_past,
:end_date_time_cannot_be_before_start_date_time
...
def start_date_time_cannot_be_in_past
    if start_date_time.present? && start_date_time < DateTime.now
        errors.add(:start_date_time, "can't be in the past")
    end
end

def end_date_time_cannot_be_before_start_date_time
    if end_date_time < start_date_time
        errors.add(:end_date_time, "can't be before start_date_time")
    end
end


We need to add test files for the Comment and Sport models:
rails g rspec:model Comment
rails g rspec:model Sport

In the spec/factories/comments.rb file:
FactoryBot.define do
    factory :comment do
        user
        content { Faker::Lorem.paragraph }
    end
end

In the spec/factories/sports.rb file:
FactoryBot.define do
    factory :sport do
        name { Faker::Lorem.word}
    end
end

We'll get pending messages because we don't have tests in comment_spec.rb and sport_spec.rb
Everything else passes. (We can take out the pending info in those two files.)


# testing the events controller
We'll start by generate a test file:
rails g rspec:request Events

Then in the spec/requests/events_spec.rb file:
require 'rails_helper'

RSpec.describe "Events", type: :request do
    describe "GET /events" do
        it 'returns a response with all the events' do
            create(:event)
            get '/events'
            expect(response.body).to eq(Event.all.to_json)
        end
    end

    describe "GET /event" do
        let (:event) { create(:event) }

        it "returns a response with the specified event" do
            get "/events/#{event.id}"
            expect(response.body).to eq(event.to_json)
        end
    end

    describe "POST /events" do
        let(:user) { create(:user) }
        let(:sport) { create(:sport) }

        before do
            event_attributes = attributes_for(:event, user_id: user.id, sport_ids: [sport.id])
            post "/events", params: event_attributes
        end

        it 'creates a new event' do
            expect(Event.count).to eq(1)
        end

        it 'return successful response' do
            expect(response).to be_successful
        end
    end

    describe "PUT /events/:id" do
        let (:event) { create(:event) }

        before do
            put "/event/#{event.id}", params: {title: "New Title"}
        end

        it 'updates an event' do
            event.reload
            expect(event.title).to eq("New Title")
        end
    end

    describe "DELETE /events/:id" do
        let (:event) { create(:event) }

        before do
            delete "/events/#{event.id}"
        end

        it 'deletes an event' do
            expect(Event.count).to eq(0)
        end

        it 'destroys event participants' do
            event_participants = EventParticipant.where(event_id: event.id)
            expect(event_participants).to be_empty
        end
        [German moved this to the event_spec.rb to test in the model]

    end
end

context "destroy related associations" do
    it 'destroys event participants' do
        event = create(:event)
        event_id = event.id
        event.destroy
        event_participants = EventParticipant.where(event_id: event.id)
        expect(event_participants).to be_empty
    end
end

If we run bundle exec rspec now we'll get errors because we haven't created the events controller.
rails g controller events
[n to not overwrite the test file we already have]

In the config/routes.rb file:
resources :events

Then in the events_controller.rb file we'll write methods:

class EventsController < ApplicationController
    before_action :set_event, only: [:show, :update, :destroy]

    def index
        events = Event.all
        render json: events, status: :ok
    end

    def show
        render json: @event, status: :ok
    end

    def create
        event = Event.new(event_params)

        if event.save
            render json: event, status: :created
        else
            render json: event.errors, status: :unprocessable_entity
        end
    end

    def update
        if @event.update(event_params)
            render json: @event, status: :ok
        else
            render json: @event.errors, status: :unprocessable_entity
        end
    end

    def destroy
        if@event.destroy
            render json: nil, status: :ok
        else
            render json: @event.errors, status: :unprocessable_entity
        end
    end

    private
    
    def set_event
        @event = Event.find(params[:id])
    end

    def event_params
        params.permit(:title, :content, :start_date_time, :end_date_time, :guests, :user_id, :sport_ids => [])
    end
end


# setting up BCrypt to hash user passwords
We'll want to include tests in the spec/models/user_spec.rb file.

In the context 'Validations tests'

it 'is invalid when password is nil' do
    user = build(:user, password: nil)
end

it 'is invalid when password_confirmation is nil' do
    user = build(:user, password_confirmation: nil)
end

it 'hashes the password' do
    user = create(:user)
    expect(user.password_digest).not_to eq 'password'
end

We'll also need to add password and password_confirmation to the user factory:
password { 'password' }
password_confirmation { 'password' }

We'll add the bcrypt gem to the Gemfile:
gem 'bcrypt'

Then run bundle install

In the app/models/user.rb file we need to include:
has_secure_password

Then we need to create a migration file:
rails g migration AddPasswordDigestToUsers password_digest:string

Then we run rails db:migrate

After running bundle exec rspec, we get two failures and need to look at the user controller.

We need to add :password, :password_confirmation to the user_params.

When we create a new user, it'll match the password to the password_confirmation and send back a password_digest as part of the response.


# login action and JWTs
We'll create a controller for users to login
First, we'll create a test file:
rails g rspec:request Sessions

In the spec/requests/sessions_spec.rb file:
require 'rails_helper'

RSpec.describe "Sessions", type: :request do
    describe "POST /login" do

        let(:user) { create(:user) }

        it 'authenticates the user and returns a success response' do
            post '/login', params: { username: user.username, password: user.password }
            expect(response).to have_http_status(:success)
            expect(JSON.parse(response.body)).to include('token')
        end

        it 'does not authenticate the user and returns an error' do
            post '/login', params: { username: user.username, password: "wrong_password" }
            expect(response).to have_http_status(:unauthorized)
        end
    end
end

We need to define the routes in the confic/routes.rb file (we'll add a scope for all the paths that are at the root level, the root path):
scope '/' do
    post 'login', to: 'sessions#create'
end

Then we'll generate a sessions controller with the create action:
rails g controller sessions create
[n to not overwrite our test file]

The way we're setting this up with JWTs we don't need to store tokens in the database. 

We need to add gem 'jwt' to the Gemfile.
Then run bundle install.

In the app/controllers/sessions_controller.rb file:
class SessionsController < ApplicationController
    def create
        user = User.find_by(username: params[:username])

        if user&.authenticate(params[:password])
            token = jwt_encode(user_id: user.id)
            render json: {token: token}, status: :ok
        else
            render json: {error: "unauthorized"}, status: :unauthorized
        end
    end

    private

    def jwt_encode(payload, exp = 24.hours.from_now)
        payload[:exp] = exp.to_i
        JWT.encode(payload, Rails.application.secret_key_base)
    end
end


# Authenticating User Requests
In the spec folder we create a folder called support, and in that folder we'll create a file, auth_helpers.rb

This will hold a module where we'll define a method auth_token_for_user:
module AuthHelpers
    def auth_token_for_user(user)
        JWT.encode({user_id: user.id}, Rails.application.secret_key_base)
    end
end

We want to make sure our spec files have access to this module.
In the rails_helper.rb file:
require 'support/auth_helpers'

And in the RSpec.config (in the same rails_helper.rb file) we need to add:
config.include AuthHelpers, type: :request

Then in the spec/requests/users_spec.rb file:
[We'll require a token for each other actions except creating a user]
...
describe "GET /users" do
...
let(:token) { auth_token_for_user(user)}
...

before do
...
get "/users", headers: { Authorization: "Bearer #{token}" }

We can copy and paste those two lines of code into the show action also.
Also, the update and delete.

Then we'll add those bits of code to the actions for posts (in the posts_spec.rb file).
We'll also need to make sure we have a user for every time we have a token. Copying the line:
let(:user) {create(:user)}

We need to include tests for what happens when a user doesn't have a token.

First, in the application_controller.rb file:
class ApplicationController < ActionController::API
    def authenticate_request
        header = request.headers['Authorization']
        header = header.split(' ').last if header

        begin
            decoded = JWT.decode(header, Rails.application.secret_key_base).first
            @current_user = User.find(decoded["user_id"])
        rescue JWT::ExpiredSignature
            render json: {error: "Token has expired"}, status: :unauthorized
        rescue JWT::DecodeError
            render json: {error: "unauthorized"}, status: :unauthorized
        end
    end
end

Then in the users_controller.rb file:
before_action :authenticate_request, only: [:index, :show, :update, :destroy]

In the posts_controller.rb file:
before_action :authenticate_request

We also want to add tokens for events.
In the events_controller.rb file:
before_action :authenticate_request

Then we need to add that to our spec/requests/events_spec.rb file:
    let(:user) {create(:user)}
    let(:token) { auth_token_for_user(user)}

    , headers: { Authorization: "Bearer #{token}" }


# Identifying Current User through Requests
In the application_controller.rb file we have
@current_user = User.find(decoded["user_id"])

This means every specific request will have a current user if the token is valid.

In the posts_controller.rb in the post_params, right now we're requiring the :user_id.
However, since we're requiring the token with these requests, we don't need to include the :user_id there. We know who the user is because they have a token.

In the update method, we can leave that how it is, with the @post.update(post_params), but the create method we'll do differently:
post = @current_user.posts.new(post_params)

In the posts_spec.rb file we'll change from user_id: nil to content: nil (in the test context "with invalid params" for the POST post). We also don't need a user_id in the post_attributes for the context "with valid params".

In the events_spec.rb we'll also remove all the places we used user_id.
(In the event attributes for the POST events.)

If we run bundle exec rspec, the tests will fail saying we didn't include a user.
So in the events_controller.rb file, we can take user_id out of the event_params.
Then in the create method we'll use:
event = @current_user.created_events.new(event_params)


# Serializing Data with Blueprinter
We install the blueprinter gem in the Gemfile:
gem 'blueprinter'
Then run bundle install

Blueprinter allows us to choose what we send back in the response from the server.

We'll create a blueprint for user:
rails g blueprinter:blueprint user

Then in the app/blueprints/user_blueprint.rb file:
class UserBlueprint < Blurprinter::Base
    identifier :id

    view :normal do
        fields :username
    end
end

This will create a default view will include the username field and disregard all the others.

Then we can use that in the users_controller.rb file:
def show
    render json: UserBlueprint.render(@user, view: :normal), status: 200
end

Next we'll create a blueprint for a user's profile:
rails g blueprinter:blueprint profile

In the app/blueprints/profile_blueprint.rb file:
class ProfileBlueprint < Blueprinter::Base
    identifier :id
    fields :bio

    view :normal do
        association :user, blueprint: UserBlueprint, view: :profile
    end
end

Then we'll need to add the profile view to the user_blueprint.rb:

view :profile do
    association :location, blueprint: LocationBlueprint
    association :posts, blueprint: PostBlueprint, view: :profile do |user, options|
        user.posts.order(created_at: :desc).limit(5)
    end

    association :events, blueprint: EventBlueprint, view: :profile do |user, options|
        user.events.order(start_date_time: :desc).limit(5)
    end
end

Then we'll create three separate blueprints for the location, post, and event.

rails g blueprinter:blueprint location

In the app/blueprints/location_blueprint.rb file:
identifier :id
fields :zip_code, :city, :state, :country, :address

rails g blueprinter:blueprint post

In the app/blueprints/post_blueprint.rb file:
identifier :id

view :profile do
    fields :content, :created_at
end

rails g blueprinter:blueprint event

In the app/blueprints/event_blueprint.rb file:
identifier :id

view :profile do
    fields :content, :start_date_time, :end_date_time, :guests, :title
end

Then we'll genreate a controller for profiles with the show action:
rails g controller profiles show

In the config/routes.rb file:
[take out the default get 'profiles/show'], then above resources :posts we'll add:
scope :profiles do
    get ':username', to: "profiles#show"
end

Then in the profiles_controller.rb:
def show
    user = User.find_by(username: params[:username])
    profile = user.profile
    render json: ProfileBlueprint.render(profile, view: :normal), status: :ok
end


When a user gets created we want to make sure a profile gets created. 
In the models/user.rb file:
after_create :create_profile

In the profiles_controller we'll add:
before_action :authenticate_request