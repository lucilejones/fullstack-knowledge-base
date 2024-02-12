Creating a simple Rails API for a todo list app

# set up the rails api:
rails new todo_list_api --api

# Creating tests with rspec
remove all mini-test files:
rm -rf test/

Include in the Gemfile (in the development, test section):
gem 'rspec-rails'
gem 'factory_bot_rails'
gem 'faker'

Then run bunlde install, then rails generate rspec:install
Then make sure to include require 'faker' in the rails_helper.rb file

In the rails_helper.rb file we also need to include:
RSpec.configure do |config|
    config.include FactoryBot::Syntax::Methods
end

Then we'll create an rspec file for the todo model:
rails generate rspec:model Todo

Then in spec/factories/todos.rb file:
FactoryBot.define do
  factory :todo do
    body { Faker::Lorem.sentence }
    is_completed { false }
  end
end

Then we create tests for the todo model. In the spec/models/todo_spec.rb file:
require 'rails_helper'

RSpec.describe Todo, type: :model do

  context "valid attributes" do
    it "is valid with valid attributes" do
      todo = build(:todo)
      expect(todo).to be_valid
    end
  end

  context "invalid attributes" do
    it "is invalid without a body" do
      todo = build(:todo, body: nil)
      todo.valid?
      expect(todo.errors[:body]).to include("can't be blank")
    end

    it "is invalid when body is too long" do
      todo = build(:todo, body: "a" * 256)
      todo.valid?
      expect(todo.errors[:body]).to include("is too long (maximum is 255 characters)")
    end

    it "is invalid without a is_completed" do
      todo = build(:todo, is_completed: nil)
      todo.valid?
      expect(todo.errors[:is_completed]).to include("is not included in the list")
    end
  end
end

Then we'll create a test for the todo controller.
rails generate rspec:request Todos

We'll define a structure for our response and then write tests for the index, show, create, update, and destroy actions.

In the spec/requests/todos_spec.rb file:
require "rails_helper"

RSpec.describe "Todos", type: :request do
  let(:expected_todo_structure) do
    {
      "id"=> Integer,
      "body" => String,
      "is_completed" => [TrueClass, FalseClass],
    }
  end
end

The variable exepcted_todo_structure is a hash the defines the structure of the response we expect from the API: id, body, and is_completed.

Then we add tests for all the actions:
describe "GET /index" do
    before do
      create_list(:todo, 10)
      get "/todos"
      @body = JSON.parse(response.body)
    end

    it "returns todos" do
      @body.each do |todo|
        expect(todo.keys).to contain_exactly(*expected_todo_structure.keys)
      end
    end

    it "returns http success" do
      expect(response).to have_http_status(:success)
    end

    it 'does not return empty if todos exist' do
      expect(@body).not_to be_empty
    end

    it 'returns 10 todos' do
      expect(@body.size).to eq(10)
    end
  end

  describe "GET /show" do
    let (:todo_id) { create(:todo).id }

    before do
      get "/todos/#{todo_id}"
      @body = JSON.parse(response.body)
    end

    it 'checks for the correct structure ' do
      expect(@body.keys).to contain_exactly(*expected_todo_structure.keys)
    end

    it "returns http success" do
      expect(response).to have_http_status(:success)
    end
  end

  describe "POST /create" do

    before do
      post "/todos", params:  attributes_for(:todo)
      @body = JSON.parse(response.body)

    end

    it 'checks for the correct structure ' do
      expect(@body.keys).to contain_exactly(*expected_todo_structure.keys)
    end

    it 'count of todos should increase by 1' do
      expect(Todo.count).to eq(1)
    end

    it "returns http success" do
      expect(response).to have_http_status(:success)
    end
  end

  describe "PUT /update" do
    let (:todo_id) { create(:todo).id }

    before do
      put "/todos/#{todo_id}", params: { body: 'updated body' }
      @body = JSON.parse(response.body)
    end

    it 'checks for the correct structure ' do
      expect( @body.keys).to contain_exactly(*expected_todo_structure.keys)
    end

    it 'checks if the body is updated' do
      expect(Todo.find(todo_id).body).to eq('updated body')
    end

    it "returns http success" do
      expect(response).to have_http_status(:success)
    end
  end

  describe "delete /destroy" do
    let (:todo_id) { create(:todo).id }

    before do
      delete "/todos/#{todo_id}"
    end

    it 'decrements the count of todos by 1' do
      expect(Todo.count).to eq(0)
    end

    it "returns http success" do
      expect(response).to have_http_status(:success)
    end
  end

To run the tests: bundle exec rspec

# Passing Todo Model Tests
To run the models tests: bundle exec rspec spec/models/todo_spec.rb

We need to create a new model for the todo:
rails g model Todo body:string is_completed:boolean
[n to skip overwriting the test files]

rails db:migrate

We'll add validations to the Todo model:
class Todo < ApplicationRecord
  validates :body, presence: true, length: { maximum: 255 }
  validates :is_completed, inclusion: { in: [true, false] }
end

# Passing Todo Controller Tests
In the routes file:
Rails.application.routes.draw do
  resources :todos
end

Then we generate a controller:
rails g controller todos

Then we define a serializer for the Todo model because our tests require a specific structure for the response:
First we need to include the gem 'blueprinter' in the Gemfile and run bundle install
rails g blueprinter:blueprint todo

This will create a new file app/blueprints/todo_blueprint.rb
In that file we'll add:
class TodoBlueprint < Blueprinter::Base
  identifier :id

  view :normal do
    fields :body, :is_completed
  end
end

Here we're defining the fields we want to serialize and defining the indentifier which is the id.

Then in the todos controller:
class TodosController < ApplicationController
  before_action :set_todo, only: [:show, :update, :destroy]

  def index
    todos = Todo.all
    render json: TodoBlueprint.render(todos, view: :normal), status: :ok
  end

  def show
    render json: TodoBlueprint.render(@todo, view: :normal), status: :ok
  end

  def create
    todo = Todo.new(todo_params)

    if todo.save
      render json: TodoBlueprint.render(todo, view: :normal), status: :created
    else
      render json: todo.errors, status: :unprocessable_entity
    end
  end

  def update

    if @todo.update(todo_params)
      render json: TodoBlueprint.render(@todo, view: :normal), status: :ok
    else
      render json: @todo.errors, status: :unprocessable_entity
    end
  end

  def destroy
    if @todo.destroy
      render json: nil, status: :ok
    else
      render json: @todo.errors, status: :unprocessable_entity
    end
  end

  private

  def set_todo
    @todo = Todo.find(params[:id])
  end
  def todo_params
    params.permit(:body, :is_completed)
  end
end

Then when we run the tests we should pass all of them.

# Full stack overview
