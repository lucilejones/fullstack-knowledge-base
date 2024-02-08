# Pickup sports api ERD
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
(We include :posts_index at the top in the before_action - since we're using the @user from the set_user method)
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


