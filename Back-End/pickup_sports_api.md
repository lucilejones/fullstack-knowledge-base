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

