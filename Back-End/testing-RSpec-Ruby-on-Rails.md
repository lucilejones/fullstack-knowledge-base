# About Testing
Testing verifies our code is working as expected
-helps us find bugs
-helps us write better code
-helps refactoring go more smoothly
-helps with documenting code
-helps with collaborating
-helps us build better poducts


# Test Driven Development
Involves writing tests before writing code:
1. write a test
2. run the test - it will fail
3. write code to make the pass test
4. refactor code


# Types of Testing
-unit testing - tests individual units of code
-integration testing - tests how different units of code work together
-functional testing - tests the functionality of the app
-acceptance testing - tests the acceptance criteria of the app
-regression testing - tests to ensure the new code doesn't break the existing code
-performance testing - tests the performance of the app
-security testing - tests the security of the app


# Testing in Ruby on Rails
By default Ruby on Rails ships with a testing framework called Minitest. This one is very simple and easy to use, but isn't as powerful as RSpec.

To generate a new project with RSpec instead of Minitest:
rails new project-name -T --api

If we have a pre-existing project, we can remove the test files with:
rm -rf test/

Then we add the following gems to our Gemfile:
group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mingw x64_mingw ]
  gem 'rspec-rails'
  gem 'factory_bot_rails'
end

We're adding the gems to the development and test groups. We only want to use these gems in teh development and test environments and not in production. 

gem 'rspec-rails' - the RSpec testing framework made for Ruby on Rails
gem 'factory_bot_rails' - this gem allows us to create factories for our tests; we can create test data

Then we run (installs the gem files):
bundle install

Then we run (generates the RSpec configuration files):
rails generate rspec: install

This will generate the following files:
create  .rspec
create  spec
create  spec/spec_helper.rb
create  spec/rails_helper.rb

In order to use factory-bot (and its methods) in our tests, we need to add the following to our rails_helper.rb file:
...
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
...
end


Unit Testing Models
We'll be using test driven development.

Create a new folder under spec called models.
Create a file called post_spec.rb:
require 'rails_helper'

RSpec.describe Post, type: :model do
  context 'with valid attributes' do
    it 'is valid' do
      post = Post.new(title: 'My First Post', content: 'Hello, world!')
      expect(post).to be_valid
    end
  end
end

Then (to see the test) we run:
bundle exec rspec spec/models/post_spec.rb

Or (to run all tests) we run:
bundle exec rspec

We should get output like the following:
An error occurred while loading ./spec/models/post_spec.rb.
Failure/Error:
  RSpec.describe Post, type: :model do
    context 'with valid attributes' do
      it 'is valid' do
        post = Post.new(title: 'My First Post', content: 'Hello, world!')
        expect(post).to be_valid
      end
    end
  end

NameError:
  uninitialized constant Post

  RSpec.describe Post, type: :model do
                 ^^^^
# ./spec/models/post_spec.rb:5:in `<top (required)>'
No examples found.


Finished in 0.00006 seconds (files took 1.51 seconds to load)
0 examples, 0 failures, 1 error occurred outside of examples

The steps of what we're doing:
1. We're requiring the rails_helper file. It contains the configuration for RSpec and Rails. It also includes the spec_helper file.
2. RSepc.describe method - takes two arguments: the name of the class we're testing and teh type of test we're running. In this case, we're testing the Post model and running a model test.
3. context - a way to group tests together; in this cae, we're grouping tests that have valid attributes.
4. We define a test - we use the it method; in this case we're testing that a post is valid. We create a new post with a title and content and we're expecting it to be valid.
5. Then we create the correct expectation; in this case, we're expecting the post to be valid using the be_valid matcher.
6. Then we run rspec in the terminal to run the test.

Since in TDD we write the test first, we expect this test to fail.
Then we'll write code to make the test pass.


Run bundle exec rails generate model Post title:string content:text

We'll get a conflict because it's trying to generate a test but we already have one. 
Enter n to skip this.

Then we need to run the migration:
bundle exec rails db:migrate

And then we'll run the test again:
bundle exec rspec spec/models/post_spec.rb

We should get the following output:
.

Finished in 0.06584 seconds (files took 3.53 seconds to load)
1 example, 0 failures

Now the test is passing. The test creates a new post with a title and content. Since we hace title and content attributes in our Post model, the test passes.

In our post_spec.rb file we'll add another test:

  context 'with invalid attributes' do
    it 'is invalid without a title' do
      post = Post.new(content: 'Hello, world!')
      expect(post).to be_invalid
    end
  end

This context is grouping tests that have invalid attributes.

We're creatig a new post without a title, and we're expecting it to be invalid. This will help make sure our validations are working as expected.

To pass this test we'll need to add a validation to our Post model:
class Post < ApplicationRecord
  validates :title, presence: true
end


Adding a test to ensure a post without content is invalid.
In the post_spec.rb file:
...
    it 'is invalid without content' do
      post = Post.new(title: 'My First Post')
      expect(post).to be_invalid
    end
...

In the post.rb file:
validates :content, presence: true


Unit Testing Controllers

To create a test for the PostsController, first we'll make a new folder in spec called requests, then a file called posts_spec.rb
In that file we'll include:
require 'rails_helper'

RSpec.describe 'Posts API', type: :request do
  describe 'POST /posts' do
    let(:valid_attributes) { { title: 'My first post', content: 'Content of the post' } }

    context 'when the request is valid' do
      it 'creates a new post' do
        expect {
          post '/posts', params: { post: valid_attributes }
        }.to change(Post, :count).by(1)

        expect(response).to have_http_status(201)
        expect(json['title']).to eq('My first post')
        expect(json['content']).to eq('Content of the post')
      end
    end

  end

  def json
    JSON.parse(response.body)
  end
end

Again, we include the rails_helper file.
We use the RSpec.describe method to describe the PostsController and specify it is a request test.

Then in the code block we define a test for the POST /posts endpoint. We use let to define a variable, valid_attributes, which is a hash with the title and content attricutes. 

Then the context groups tests that have valid attributes. 
Then the it method defines a test that includes an argument that describes what the test does; in this case, we're testing that a new post is created.

expect - what we expect to happen (a new post to be created)
{ post '/posts', params: { post: valid_attributes } } - the code we expect to run
to - the matcher; we're expecting the code to change the Post count by 1.

Then we chain the expect method with the to method; we're expecting the response to have a status of 201 (the status for a successful POST request)

Then we list what we expect the title and content to equal (matching the valid_attributes hash).

Then we have a helper method that parses the response body into JSON.

When we run our test we expect it to fail.
exec rspec spec/requests/posts_spec.rb

Then we'll generate a contoller:
rails g controller Posts
(We don't need to generate a model because we already have one.)
(It might prompt us to overwrite the posts_controller.rb file. Enter n to skip this because we already have a spec file for this.)

Then we add the following code to our PostsController:
class PostsController < ApplicationController
  def create
    post = Post.new(post_params)
    if post.save
      render json: post, status: :created
    else
      render json: post.errors, status: :unprocessable_entity
    end
  end

  private

  def post_params
    params.require(:post).permit(:title, :content)
  end
end

Then in the config/routes.rb file:
  resources :posts

Then when we run the test again, it passes.
bundle exec rspec spec/requests/posts_spec.rb


It's good practice to separate our tests into multiple tests. This makes them more readable and easier to debug. In the spec/requests/posts_spec.rb file:
require 'rails_helper'

  RSpec.describe 'Posts API', type: :request do
    describe 'POST /posts' do
      let(:valid_attributes) { { title: 'My first post', content: 'Content of the post' } }

    context 'when the request is valid' do
      before { post '/posts', params: { post: valid_attributes } }

      it 'creates a new post' do
        expect {
          post '/posts', params: { post: valid_attributes }
        }.to change(Post, :count).by(1)
      end

      it 'returns status code 201' do
        expect(response).to have_http_status(201)
      end

      it 'returns the created post' do
        expect(json['title']).to eq('My first post')
        expect(json['content']).to eq('Content of the post')
      end

      it 'saves the post with the correct attributes' do
        post = Post.last
        expect(post.title).to eq('My first post')
        expect(post.content).to eq('Content of the post')
      end
    end
  end

  def json
    JSON.parse(response.body)
  end
end


(The before method will run that code before each test. Here we're making a POST request to the /posts endpoint with the valid_attributes hash)

Then we can create a test for the GET /posts endpoint.
In the same spec/requests/posts_spec.rb file:
...
  describe 'GET /posts' do
    let!(:posts) { create_list(:post, 10) } # creating 10 posts using Factory Bot

    before { get '/posts' }

    it 'returns posts' do
      expect(response).to have_http_status(200)
      expect(json).not_to be_empty
      expect(json.size).to eq(10)
    end

    it 'returns posts with the correct structure' do
      # Assuming each post has 'title' and 'content'
      json.each do |post|
        expect(post).to include('title', 'content')
      end
    end
  end
...

We create a new describe block for the GET /posts endpoint.

Then we use the let! method to create 10 posts using factory-bot
create_list is a factory bot method that creates a list of objects; in this case a list of 10 posts.

If we open the spec/factories/posts.rb file we can see the factory for our posts:
FactoryBot.define do
  factory :post do
    title { "MyString" }
    content { "MyText" }
  end
end

Factories allow us to define a common setup for creating objects. We define it once in a factory and then reuse it across multiple tests.
They also abstract away the complexities of object creation and allow tests to be cleaner and more focused on the behavior being tested rather than the setup.

