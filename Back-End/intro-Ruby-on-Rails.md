# Full Stack Web Development
-frontend development, also known as client-side development
-HTML, CSS, JS; for a user to see and interact directly with them
-backend development, also called server-side development
-focus on databases, scripting, and website architecture
-also server management and APIs

System design
-focus on the architecture of the system: optimizing code and database queries, employing caching strategies and managing servers and networks (also scaling over time)

Other aspects of web development:
Design (UI and UX)
Testing
Project Management
DevOps
Security

Common Technologies
Frontend:
HTML/CSS, JS
Frameworks: Angular, ReactJS, Vue
Libraries: JQuery, Bootstrap

Backend:
Node.js, Ruby, Python, PHP, Java
Frameworks: Express (JS), Django (Python), Ruby on Rails (Ruby)
Databases: MySQL, MongoDB, Oravle, SQLServer
Server management: AWS, Heroku, others

DevOps and Version Control:
Git
Docker, Jenkins, Kubernetes

Testing and Debugging:
JUnit (Java), Jest (JS)
Chrome DevTools, VS Code Debugger


# Understanding the Internet and Web Servers
The internet is a network of globally interconnected computers and servers. Data transmission follows a system of standard protocols (a set of rules). The most comon protocol is TCP/IP (Transmission Control Protocol/Internet Protocol).

Clients - request data from servers
Servers - devices that store and send data to clients

Web Servers - powerful computers that store website files and serve them to users on request; run software like Apache or Nginx

Clients - typically web browsers like Chrome or Firefox; request info from servers.

Request-Response Cycle - the browser sends a request to teh server which then responds with the site's files

IP address - a unique identifier assigned to every device connected to the internet

DNS - Domain Name System; translate domain names (like google.com) into IP addresses

Web Hosting - a service that makes websites accessible via the world wide web

Deployment - making a website available to the public; uploading files to a web server and making them accessible via the world wide web.

HTTP
Hypertext Transfer Protocol
The backbone of data communication on the web. A client sends an HTTP request to the server, the server processes the request and sends back a response. The response contains a status line and a message body (which is often teh request resource, like an HTML doc).

HTTP methods:
GET, POST, PUT, DELETE
HEAD - similar to GET, but asks for a response identical to a GET without the response body

A resource is described as a location or object capable of being identified by a URI (uniform resource identifier). Examples of resources: HTML documents, images, videos, style sheets. A URI is a string of character that unambiguously identifies a particular resource.

HTTP Status Codes
Informational responses (100-199)
Successful responses (200-299)
Redirection messages (300-399)
Client error responses (400-499)
Server error responses (500-599)


# REST and RESTful APIs
There are different ways to construct a backend: SOAP, XML-RPC, REST, GraphQL, custom structured.
The most common (and widely used) way to build APIs is REST: Representational State Transfer

REST is a set of guidelines based on the idea that everything is a resource and can be accessed by a unique identifier (URI).
REST is stateless, meaning each request can be processed independently of the previous one.
This makes REST APIs scalable and eay to maintain.

What makes an API Restful:
Resources - REST APIs are resource-based. (Example, user is a resource and can be accessed using a URI like /uesr/1)
HTTP Methods
Status Codes
Stateless
Caching

Endpoint or routes are the most important part.

URLs
Uniform Resource Locator - a string of characters that identifies a particular resource
Made up of three parts: the protocol, the domain name, the path

Query parameters: pass additional info to the server; often used to filter data

RESTful endpoints
Not just URLS; they are also HTTP methods.


# Ruby on Rails Intro
A web application framework written in Ruby. One of the most popular.
Even though it's a framework, it's a gem (a package that contains Ruby code).
Similar to a Node.js module or a Python package.

Rails is a Model-View-Controller (MVC) framework. The MVC pattern is a software pattern that separates the representation of info from the user's interaction with it. 
We organize code into three distinct parts: models, views, controllers.

Models: responsible for managing the data of the app; the interface between the app and the database; also responsible for validating data and enforcing business rules

Views: responsible for displaying data to the user; the interface between the app and the user; also responsible for handling user input

Controllers: responsible for handling requests and generating responses; the interface between the app and the web server; also responsible for handling business logic

Rails is a full-stack framework, so it provides everything we need to build an app, both frontend and backend. Includes a router, a controller, a model, a view, and a database.
Includes other features, like a built-in server, a built-in testing framework, and a built-in debugging framework. 

Advantages:
Convention over configuration - Rails makes assumptions about what every developer needs allowing us to write less code
Full-stack framework
Active Record - Rails uses a database abstraction layer allowing us to interact with the database using Ruby objects instead of SQL queries
Gems - there are gems for authentication, authorization, and pagination
Community - has a large and active community
Open Source - anyone can contribute to it; fix bugs, add features, improve documentation
Fast Development

Disadvantages:
Not as flexible
Performance - Rails can be slower than other frameworks
Learning Curve - can be challenging for beginners
Memory Usage - Rails apps can consume more memory compared to other frameworks

Installing rvm:
https://github.com/rvm/ubuntu_rvm

Installing Ruby:
rvm install ruby 3.2.0

Installing Ruby on Rails:
in the WSL2 Ubuntu terminal:
gem install rails

This will install Rails globally on the computer.
This will give us access to the Rails CLI. Allows us to create new Rails projects, generate files, and run automated commands

We'll be using Rails 7

Creating a new Rails Project:
rails new <project_name>
(use snake_case separating with an underscore)
By default, running this command will create a new Rails project confiured and setup as a full stack project.

If we only want to create a Rails API (just the backend):
rails new my_first_rails_project --api
This will create a new Rails project configured and setup to be used as a RESTful API.

Rails creates a git repository for us. 

The app folder is where we'll put most of our code.
The app has subfolders:
-channels: contains the Action Cable channels for our app. Action Cable is a framework for real-time communication over WebSockets.
-controllers: contains the controllers for our app. Controllers are responsible for handling requests and generating responses.
-jobs: contains the Active Job jobs for our app. Active Job is a framework for declaring jobs and making them run on a variety of queuing backends.
-mailers: contains the mailers for our app. Mailers are responsible for sending emails.
-models: contains the models for our app. Models are responsible for managing the data of the app.
-views: contains the views for our app. Views are responsible for displaying data to the user; however, because we're just building a RESTful API, we will only use this folder to craft email templates.

The bin folder contains the rails executables, used to run commands like rails server and rails console.
-the rails server command starts the Rails server; the Rails server is a web server that runs our app
-the rails console command starts the Rails console; the Rails console is an interactive Ruby shell that allows us to interact with our app and the database

The config folder contains configuration files for our app, like routes.rb, database.yml, application.rb
-the routes.rb file defines the routes for our app. Routes are used to map URLs to controllers and actions
-the database.yml file configures the database for our app. Rails uses SQLite3 by default
-the application.rb file configures the app, like the default locale, the default time zone, and the default encoding

The db folder contains the database schema and migrations for our app
-by default Rails will use SQLite3; it's lightweight and compatible with multiple devices

The lib folder contains the code that's shared between different parts of our app. For example, custom validators.

The log folder contains the log files for our app. This is where we find info about what's happening in the app when it's running. We can find this while running the server or when we deploy the app to a server.

The public folder contains the static files for our app: images, JS, CSS

The storage folder contains files uploaded by users.

The test folders contains the tests for our app.

The tmp folder contains temporary files for our app.

The vendor folder contains third-party code that is not managed by Bundler.


Using the Rails CLI to generate a model

Models are responsible for managing the data of the application; they're the interface between the app and the database. 
Ruby on Rails has a built-in ORM (Object Realtional Mapping) called Active Record, which allows us to interact with the database using Ruby objects instead of SQL queries.

Models also validate data and enforce business rules (like the rule that a user can't have the same email address as another user or that a product can't have a negative price)

To create a model we can use the rails generate model command. We cd into the project folder, and then run
rails generate model User name:string email:string

rails generate model - the rails command used to create a model

User - the name of the model; convention to use singular nouns for model names

name:string email:string - attributes of the model.
These define the columns of the database table. The name of the attribute and the type.

This creates a new model called User (file path app/models/user.rb).
It will also create a migration file for the User model.

A migration is a Ruby class used to make changes to the database; a way to update the database schema. We can use migrations to create tables, add columns, and remove columns. 


Migrations
We can look at the migration file that was gnerated for us.
Navigate to the db/migrate folder.
The file name should look similar to this: 20240110170144_create_users
And in the file should look similar to this:
class CreateUsers < ActiveRecord::Migration[7.0]
  def change
    create_table :users do |t|
      t.string :name
      t.string :email

      t.timestamps
    end
  end
end

class CreateUsers < ActiveRecord::Migration[7.0]
-name of the migration class; it's convention to use the name of the model followed by the name of the migration.
-Here, the model is User and the migration is CreateUsers; the migration class inherits from teh ActiveRecord::Migration class. This is the base class for all migrations.

change method
-used to make chages to the database; is a method defined by the ActiveRecord::Migration class

create_table method
-used to create a new table in the database; is a method defined in teh ActiveRecord::ConnectionAdapters::SchemaStatements class

:users
-name of the table; convention to use the plural form of the model name

t.string:name
t.string:email
-names of the columns; convention to use the singular form of the attribute name

t.timestamps
-a shortcut for creating two columns called created_at and updated_at
-these are used to store the date and time when a record is created and updated


Common data types to use for our columns:
string, text, integer, float, decimal, datetime, boolean, binary, json


Migration files
These are Ruby classes used to make changes to the database; a way to update the database schema. We can create tables, add columns, remove columns.

The file name is prefixed with a timestamp, this determines the order in which migration files are applied. If we have two migration files with the same timestamp, Rails will apply them in alphabetical order. 
Migrations can depend on each other.
One migration might add a column to a table and another will remove that same column. 
Rails will apply the first migration before the second. Called a migration dependency.

We don't delete or modigy migration files. To make changes we create a new migration file.
Rails uses the migration files to determine the current state of the database. 

Since we generated the creation of the file using a Ruby on Rails command, it didn't auto create the table in the database. We need to run the migration file.
We use:
rails db:migrate

This will runall the pending migrations.
In this example, it will create the users table in the database.
We'll see something like this:
== 20231230191708 CreateUsers: migrating ======================================
-- create_table(:users)
   -&gt; 0.0018s
== 20231230191708 CreateUsers: migrated (0.0019s) =============================

If we made a mistake we can run:
rails db:rollback
This will rollback the last migration and reset the state of the migration file back to pending. It will remove the users tables from the database because that was the last migration we ran.

Then we see something like this:
== 20231230191708 CreateUsers: reverting ======================================
-- drop_table(:users)
   -&gt; 0.0018s
== 20231230191708 CreateUsers: reverted (0.0019s) =============================

[but then we'll run the rails db:migrate again because we do want the users table]

We can rollback muliple migrations by specifying the number of migrations we want to rollback.
rails db:rollback STEP=2


Schema
In the db folder, schema.rb

The schema is essentially a representation of the database.
It will include all the tables and columns.
We don't manually modify this file. It's auto generated by Rails and is used to determine the current state of teh database.
To modify the database we create a new migration file. (When we run db:migrate it will change the schema file)

Inside the schema.rb file should look something like this:
ActiveRecord::Schema.define(version: 20231230191708) do

  create_table "users", force: :cascade do |t|
    t.string "name"
    t.string "email"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

end

ActiveRecord::Schema.define
-the name of the schema class

version: 20231230191708
-the version of the schema; used to determine the current state of the database; a timestamp generated when we run the rails db:migrate command

create_table "users"
-the method used to create a new table in the database; the method is defined in the ActiveRecord::ConnectionAdapters::SchemaStatements class

t.string "name"
t.string "email"
-the method used to create a new column in the database; the method is defined in the ActiveRecord::ConnectionAdapters::TableDefinition class

t.datetime "created_at"
t.datetime "updated_at"
-a shortcut for creating two columns called created_at and updated_at
-they're used to store the data nd time when a record is created and updated

force: :cascade
-an option used to drop the table before creating it
-useful when we want to reset the state of the database

precision: 6
-an option used to specify the precision of the datetime columns
-used to store the date and time when a record is created and updated

null: false
-an option used to specify that the column cannot be null
-useful when we want to enforce business rules


The table doesn't indicate a primary key because Rails automatically creates a primary key for us. It's called id and is an integer.
The id is also auto-incremented: every time we create a new record, the id will be incremented by one.


# Interacting with the Database with Models
Rails uses a database abstraction layer called Active Record, also referred to as an ORM (Object Relational Mapping).
We interact with the database using Ruby instead of SQL queries.
Example: we can use the User model to create, read, update, and delete users.

We can use the ORM if we define the correct files and folders in the right spot.
We have a file under app/models/use.rb.
This is where we define the model.

Right now the file just looks like this:
class User < ApplicationRecord
end

ApplicationRecord
-a class defined in the ActiveRecord::Base class; the base class for all models
-we can define methods that are used to validate data and enforce business rules

class User < ApplicationRecord
-the name of the model class; convention to use the name of the model followed by the name of the class


Model Methods

To interact with the database we can use the rails console command:
rails console
This will start the Rails console, which is an interactice Ruby shell that allows us to interact with our application and the database.
Similar to the Rails server, but is used for debugging purposes or to access the database instead of running the app.

To exit the console and go back to the terminal, we run:
exit

To create a new user:
User.create(name: "John Doe", email: "johndoe@gmail.com")

This is equivalent to running the SQL query:
INSERT INTO users (name, email) VALUES ("John Doe", "joendoe@gmail.com")

User
-the name of the model

create
-the method used to create a new record (or row); the method is defined int the ActiveRecord::Persistence class

We specify the values for both of our attributes by passing in a hash (the attribute names are keys and the attribute values are values). This is called a hash literal.

This will create the new user and return the user object.

To find the user we just created we can run:
User.find(1)

This is equivalent to the SQL query:
SELECT * FROM users WHERE id = 1

The return value is an object of the User class.
To access attributes of the user, we can use dot notation. For example:
user = User.find(1)
user.name

To update the user we can run:
user = User.find(1)
user.update(name: "Jane Doe", email: "janedoe@gmail.com")

This is equivalent to the SQL query:
UPDATE users SET name = "Jane Doe", email = "janedoe@gmail.come" WHERE id = 1

To delete a user we can run:
user = User.find(1)
user.destroy

This is equivalent to the SQL query:
DELETE FROM users WHERE id = 1

To find all users:
User.all

Equivalent to SQL query:
SELECT * FROM users

To find users with a specific name:
User.where(name: "John Doe")

Equivalent to SQL query:
SELECT * FROM users WHERE name = "John Doe"

To find users with a specific name and email:
User.where(name: "John Doe", email: "johndoe@gmail.com")


Other model class methods:
find_by - find a record by a specific attribute
count - count the number of records
first - find the first record
last - find the last record
all - find all records
where - find records by a specific attribute
where.not - find records that don't have a specific attribute
order - order records by a specific attribute
limit - limit the number of records
offset - offset the number of records


Validations
-used to validate data and enforce business rules

We can enforce validations at database level, model level, and on the frontend.

Database level: most basic level
Model level: most common level

Having both offers multiple layers of security. Database-level act as a final safeguard, while model=level provide a more user-friendly and flexible option.
Model-level might not be enough, however, when multiple applications interact with the same database and each application might have different validation rules.
Frontend validations also help reduce unnecessary server requests. 

Adding validations to the User model.
In the app/models/user.rb file:
class User < ApplicationRecord
  validates :name, presence: true
  validates :email, presence: true, uniqueness: true
end

validates :name, presense: true
This validates the name attribute; it takes two arguments - the name of the attribute and the validation rule. In this case, we use the presence validation rule, saying the attribute must be present. It cannot be nil or an empty string.

validates :email, presence: true, uniqueness: true
This has multiple validations: the email attribute needs to be present and needs to be unique.

We can create a new user with:
User.create(name: "jimmy james", email: "jimmyjames@gmail.com")

If we try to create another user with the same email, we'll get an error.
If we try to create a new user without a name, we'll get an error.


Other examples of model validators:

validates :email, format: { with: URI::MailTo::EMAIL_REGEXP }
-checks if the value of an attribute matches a given regular expression.

validates :username, length: { minimum: 5, maximum: 20 }
-ensures the length of the attribute's value is within a specified range

validates :age, numericality: { greater_than: 0 }
-ensure's the attribute's value is anumber, and can validate whether it's greater than, less than, equal, to, odd, even, etc

validates :status, inclusion: { in: %w[pending approved rejected] }
-validates the value of the attribute is included in or excluded from a given set.

validates :notes, absence: true
-validates the specified attributes are empty or null

validates :password, confirmation: true
-used mainly for passwords and emails, ensuring the two text fields receive exactly the same content

validates :terms_of_service, acceptance: true
-checks if a checkbox (like terms of service) is checked.


Custom Model Validators
In the User model, we can add:
class User < ApplicationRecord
  validate :name_cannot_contain_numbers

  def name_cannot_contain_numbers
    if name.match(/\d/)
      errors.add(:name, "cannot contain numbers")
    end
  end
end

We give a name to our custome validator (:name_cannot_contain_numbers), and then define that method below. If the name attribute contains numbers, we add an error to the name attribute.


Callbacks
-used to exectue code before or after certain events.
-example: we can send an email after a user is created. Or update a user's profile after they log in.

Before or after certain events: this is the most basic type.
Before or after certain actions: this is the most common type. 

Before callbacks act as a final sadeguard ensuring data integrity; while after callbacks provide a user=friendly and flexible way to execute code.

Example of a before callback:
class User < ApplicationRecord
  validates :name, presence: true
  validates :email, presence: true, uniqueness: true

  before_create :downcase_email

  private

  def downcase_email
    self.email = email.downcase
  end
end

We're using before_create which means the method will be called before the record is created. In this case, the email wil be downcased before the record is created. 

We can create a new user:
User.create(name: "Avery Ivery", email: "aVeRyIvery");

We know the email will be downcased before the record is created.

An example of an after callback:
User.last.email



# Intro to Associations

Associations are used to define relationships between models.
Example: we can define a one-to-one relationship between a user and a profile.
We can define a one-to-many relationshipe between a user and a post.

Using Active Record makes this really easy. We follow the naming convention and Rails takes care of the rest.

one-to-one: most basic (user to profile)
one-to-many: most commone (user to post)
many-to-many: most complex (between a user and a group)
polymorphic: most advanced - we can set up a single model that can belong to more than one model, using a single association (a comment can be associated with post or video)
self-join: most advanced (we can define a self-join relationship between a user and a friend)


One-to-one: belongs to and has one associations

We can add has_one association to our User model.
We have a user table, and we need to create a profile table.
We can run:
rails generate model Profile user:references bio:text

Profile - the name of the model

user:references - the type of the attribute; used to define a one-to-one relationship between two models. Here we're using the references type, meaning the attribute will be a reference to another model. We're referencing the User model. This will create a user_id column in the profiels table.

bio:text - the name of the attribute and type.

This will create a model file under app/models/profile.rb and a migration file under db/migrate.

In the profile.rb file
class Profile < ApplicationRecord
  belongs_to :user
end

belongs_to :user - this method adds a belongs_to association. It takes two arguments, the name of the association and the options. In this case the profile belongs to a user. This will create a user method that returns the user associated with the profile.

Then we run the migration file to create the table in the database:
rails db:migrate

In the user.rb file we'll add the following code:
class User < ApplicationRecord
  validates :name, presence: true
  validates :email, presence: true, uniqueness: true

  before_create :downcase_email

  has_one :profile

  private

  def downcase_email
    self.email = email.downcase
  end
end

has_one :profile - this method adds a one-to-one relationship. It takes two arguments, the name of the association and the options. In this case, we're using the has_one association meaning the user has one profile. This will create a profile method that returns the profile associated with the user.

Now we can create a new user with a profile:
user = User.create(name: "James Underway", email: "james_underway@gmail.com", profile: Profile.new(bio: "I am a software engineer"))


One-to-many: belongs to and has many associations

We can create a post table:
rails generate model Post user:references title:string body:text published:boolean published_at:datetime

Post - the name of the model

user:references - the type of the attribute; here we're using the references type, meaning the attribute will be a reference to another model - the User model. This will create a user_id column in the posts table.

In the app/models/post.rb file:
class Post < ApplicationRecord
  belongs_to :user
end

Then we run the migration file:
rails db:migrate

And in the app/models/user.rb file we add the following code:
class User < ApplicationRecord
  validates :name, presence: true
  validates :email, presence: true, uniqueness: true

  before_create :downcase_email

  has_one :profile
  has_many :posts

  private

  def downcase_email
    self.email = email.downcase
  end
end

has_many :posts - this method adds the one-to-many relationship; it takes two arguments, the name of the association and the options. Here, we se has_many meaning the user has many posts. This will create a posts method that returns the posts associated with the user.

We can create a new user with a post:
user = User.create(name: "Amy Underway", email: "@email.com", posts: [Post.new(title: "Hello World", body: "This is my first post", published: true, published_at: "2023-12-30 19:17:08")])

We can also do something like:
user = User.find(1)
post = Post.new(title: "Hello World", body: "This is my second post", published: true, published_at: "2023-12-30 19:17:08")
user.posts << post

Then we can find the user and their posts:
user = User.find(1)
user.posts
