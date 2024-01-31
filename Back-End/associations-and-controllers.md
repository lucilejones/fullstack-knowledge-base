# Associations in-depth

https://guides.rubyonrails.org/association_basics.html


has_one 
-used in the model that contains the primary key. (The foreign key goes on the table for the class declaring the belongs_to association.) It specifies that each instance of the model has one instance of another model. One other model has a reference to this model.
example: a User model has one Profile; in the User model - has_one :profile


belongs_to
-used in the model that is owned by another model. (This model will have the foreign key.)
example: in the Profile model - belongs_to :user

rails g model User name:string email:string
rails g model Profile user:references bio:text

app/models/user.rb
class User < ApplicationRecord
  has_one :profile
end

app/models/profile.rb
class Profile < ApplicationRecord
  belongs_to :user
end

rails db:migrate
rails c

user = User.create(name: "John", email: "test@test.com")
profile = Profile.create(bio: "I am a user", user: user)
user.profile = profile


has_many

rails g model User name:string email:string
rails g model Post title:string body:text user:references

app/models/user.rb
class User < ApplicationRecord
  has_many :posts
end

app/models/post.rb
class Post < ApplicationRecord
  belongs_to :user
end

rails db:migrate
rails c

user = User.create(name: "John", email: "test@test.com")
post = Post.create(title: "My first post", body: "This is my first post", user: user)

user.posts << post


has_many :through
(makes the association indirectly)
-when a record in a table can belong to many records in another table through a third table.
example: a user has many doctors through appointments; a doctor has many users through appointments. A many-to-many associate would be a patient has many doctors and a doctor has many patients.
-this setup makes sense if we need to work with the relationship model as an independent entity (like a rental record that has ids for cars and for renters)

rails g model Physician name:string
rails g model Patient name:string
rails g model Appointment physician:references patient:references

app/models/physician.rb
class Physician < ApplicationRecord
  has_many :appointments
  has_many :patients, through: :appointments
end

app/models/patient.rb
class Patient < ApplicationRecord
  has_many :appointments
  has_many :physicians, through: :appointments
end

app/models/appointment.rb
class Appointment < ApplicationRecord
  belongs_to :physician
  belongs_to :patient
end

rails db:migrate
rails c

physician = Physician.create(name: "Dr. Smith")
patient = Patient.create(name: "John Doe")
appointment = Appointment.create(physician: physician, patient: patient)

physician.appointments
patient.appointments


has_and_belongs_to_many
(makes the association directly)
-when a record in a table can belong to many records in another table; this association does not require a third model.
example: an application includes many assemblies and parts, with each assembly having many parts and each part appearing in many assemblies.
-this setup makes sense if we don't need to work with the relationshipw model as an independent entity (though we do need to create the joining table in the database)

rails g model Author name:string
rails g model Book title:string

The third table is the authors_books table with an author_id column and a book_id column. It will be the join table between the authors table and the books table.

We need to generate a migration file to create the authors_books table:
rails g migration CreateJoinTableAuthorBook author book

The generated migration file will look like this:
class CreateJoinTableAuthorBook < ActiveRecord::Migration[6.0]
  def change
    create_join_table :authors, :books do |t|
      # t.index [:author_id, :book_id]
      # t.index [:book_id, :author_id]
    end
  end
end

We uncomment the two lines of code (the two lines that start t.index)
and run the migration.
rails db:migrate

app/models/author.rb
class Author < ApplicationRecord
  has_and_belongs_to_many :books
end

app/models/book.rb

class Book < ApplicationRecord
  has_and_belongs_to_many :authors
end

Then any time we create an author, we can create a book for that author:
rails c
author = Author.create(name: "John Doe")
book = Book.create(title: "My first book")

author.books << book


polymorphic
-when a record in a table can belong to more than one record from other tables; a model can belong to more than one other model, on a sigle association
example: a comment can belong to a post or a comment can belong to a photo
example: a picture model belongs to either an employee model or a product model

rails g model Comment body:text commentable:references{polymorphic}
rails g model Post title:string body:text
rails g model Photo title:string

The third table is going to be a comments table with a commentable_id column and a commentable_type column. The comments table is the join table between the posts table and the photos table.

We'll generate a migration file to create the comments table:
rails g migration CreateComments

The migration file will look like this:
class CreateComments < ActiveRecord::Migration[6.0]
  def change
    create_table :comments do |t|
      t.text :body
      t.references :commentable, polymorphic: true, null: false

      t.timestamps
    end
  end
end

Then we run the migration:
rails db:migrate

And then we create the association between the comment model, the post model, and the photo model:

app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end

app/models/post.rb
class Post < ApplicationRecord
  has_many :comments, as: :commentable
end

app/models/photo.rb
class Photo < ApplicationRecord
  has_many :comments, as: :commentable
end

Then when we create a post, we can create a comment for that post:
rails c
post = Post.create(title: "My first post", body: "This is my first post")
comment = Comment.create(body: "This is my first comment", commentable: post)

post.comments

photo = Photo.create(title: "My first photo")
comment = Comment.create(body: "This is my first comment", commentable: photo)

photo.comments


self-join
-when a record in a table can belong to another record in the same table
example: a user can have many friends and a friend can be a user
example: followers and following
example: we want to store all employees in a single database model, but also want to be able to trace relationships such as between manager and subordinates

We don't necessarily need a third table; instead we can use a single table (users) with a join table that references it twice (once for each side of the relationship). In the case of users and friends, the users table holds all user records and the friendships table is used to link users together.

rails g model User name:string
rails g model Friendship user:references friend:references

The migration file will look like this:
class CreateFriendships < ActiveRecord::Migration[6.0]
  def change
    create_table :friendships do |t|
      t.references :user, null: false, foreign_key: true
      t.references :friend, null: false, foreign_key: { to_table: :users }

      t.timestamps
    end
  end
end

In the friend column we specify foeign_key: { to_table: :users } to indicate this is a self-reference to the users table.

app/models/user.rb
class User < ApplicationRecord
  has_many :friendships
  has_many :friends, through: :friendships, source: :friend
end

app/models/frienship.rb
class Friendship < ApplicationRecord
  belongs_to :user
  belongs_to :friend, class_name: 'User'
end

user1 = User.create(name: "Alice")
user2 = User.create(name: "Bob")

user1.friends << user2

user1.friends



# Controllers and Routes

Controllers
-the middlepeople between models and views
-responsible for processing requests and sending responses
-also responsible for handling the logic of the app

Routes
-responsible for mapping URLs to the controller actions
-a controller action is a method defined in the controller's class

We can creat a User model and then run the migration:
rails g model User name:string email:string

rails db:migrate

Then we create a users controller:
rails g controller Users

This creates a controller file in the app/controllers/users_controller.rb
We'll add the following code:
class UsersController < ApplicationController
end

This file is where we'll add actions.
Actions are methods defined in the controller, responsible for handling requests and sending responded.

We'll add an action called create (with the logic to create a new user with the name and email passed in as parameters):
class UsersController < ApplicationController
  def create
    user = User.new(name: params[:name], email: params[:email])
    if user.save
      render json: user
    else
      render json: { error: "Unable to create user." }
    end
  end
end

[in one place in the notes the user variable had an @ in front; in another place it didn't]

The params hash contains all the parameters that are passed in from the request.
Example: if we make a POST request with these parameters:
{
	"name": "John",
	"email": "johndoe123@gmail.com"
}
The params hash will look like this:
{
  name: "John",
 email: "johndoe123@gmail.com"
}

render
-method used to render a view or a resource as JSON

redirect_to
-method used to redirect to a different route

request
-method used to access the request object

[and other methods provided by ApplicationController]

new
-method provided by ActiveRecord; will fill attributes of the record but won't save until we call the save method

Then we check to see if the user was saved. If it was, we render the user as JSON; if it wasn't, we render an error message as JSON.
(This is the typical pattern: if an action is successful, render the resource as JSON; if the action was not successful, render an error message as JSON.)

Next we add a route to the routes file.
config/routes.rb:
Rails.application.routes.draw do
  resources :users
end

The resources method, provided by Rails, creates routes for a resource.
The following routes will get created:
| HTTP Verb | Path       | Controller#Action | Used for                    |
| --------- | ---------- | ----------------- | --------------------------- |
| GET       | /users     | users#index       | display a list of all users |
| POST      | /users     | users#create      | create a new user           |
| GET       | /users/:id | users#show        | display a specific user     |
| PATCH/PUT | /users/:id | users#update      | update a specific user      |
| DELETE    | /users/:id | users#destroy     | delete a specific user      |

Controller#ACtion is the controller and action used to handle the request.

In the routes file, we can specify the only action we'd like to use:
Rails.application.routes.draw do
  resources :users, only: [:create]

Then we can start up our Rails server with the following command:
rails s


Then we can set up a collection in Postman to test our API.

rails-postman-test

The first request is create user, the url will be http://localhost:3000/users
This is a POST request.
In the Body tab, we select raw and JSON, then pass in a JSON object:
{
  "name": "Jane Doe",
  "email": "Janedoe123@email.com"
}

After hitting send, we should get a response object back with an id, name, email, created_at, and updated_at.


Index
-the index action displays a list of all users.

In the users controller:
class UsersController < ApplicationController
  def index
    @users = User.all
    render json: @users
  end

  def create
    @user = User.new(name: params[:name], email: params[:email])
    if @user.save
      render json: @user
    else
      render json: { error: "Unable to create user." }
    end
  end
end

Then we need to add a route to the routes file:
Rails.application.routes.draw do
  resources :users, only: [:create, :index]
end

Each action name is tied to a route name.

To trigger the index action, we'll make a GET request to the /users route.


Show
-the show action gets a specific user

In the users controller:
class UsersController < ApplicationController
  def index
    @users = User.all
    render json: @users
  end

  def show
    @user = User.find(params[:id])
    render json: @user
  end

  def create
    @user = User.new(name: params[:name], email: params[:email])
    if @user.save
      render json: @user
    else
      render json: { error: "Unable to create user." }
    end
  end
end

Here we find a user by the id passed in as a parameter.

We'll add the route to the routes file:
Rails.application.routes.draw do
  resources :users, only: [:create, :index, :show]
end

To trigger the show action we'll make a GET request to /users/:id

In Postman our GET request will go to http://localhost:3000/users/1


Put
-the put action updates a specific user

In the user controller:
class UsersController < ApplicationController
  def index
    @users = User.all
    render json: @users
  end

  def show
    @user = User.find(params[:id])
    render json: @user
  end

  def create
    @user = User.new(name: params[:name], email: params[:email])
    if @user.save
      render json: @user
    else
      render json: { error: "Unable to create user." }
    end
  end

  def update
    @user = User.find(params[:id])
    if @user.update(name: params[:name], email: params[:email])
      render json: @user
    else
      render json: { error: "Unable to update user." }
    end
  end
end

Here we find a user with the id passed in as a parameter and then update that user with the name and email passed in as parameters.

Updating the routes file:
Rails.application.routes.draw do
  resources :users, only: [:create, :index, :show, :update]
end

In Postman we'll send a PUT request to http://localhost:3000/users/1

And in the Body tab, we select raw and JSON:
{
  "name": "Jennifer Doe",
  "email": "newjenndoe@email.com"
}


Destroy
-the destroy action deletes a specific user

In the users controller:
class UsersController < ApplicationController
  def index
    @users = User.all
    render json: @users
  end

  def show
    @user = User.find(params[:id])
    render json: @user
  end

  def create
    @user = User.new(name: params[:name], email: params[:email])
    if @user.save
      render json: @user
    else
      render json: { error: "Unable to create user." }
    end
  end

  def update
    @user = User.find(params[:id])
    if @user.update(name: params[:name], email: params[:email])
      render json: @user
    else
      render json: { error: "Unable to update user." }
    end
  end

  def destroy
    @user = User.find(params[:id])
    if @user.destroy
      render json: { message: "Successfully deleted user." }
    else
      render json: { error: "Unable to delete user." }
    end
  end
end

Then in the routes file, since we aren't excluding any, we can take out the only option:
Rails.application.routes.draw do
  resources :users
end

