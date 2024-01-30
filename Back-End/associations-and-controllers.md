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