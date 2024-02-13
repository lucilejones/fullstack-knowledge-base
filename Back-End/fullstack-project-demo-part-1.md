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
We're using Ruby on Rails for the backend and Angular for the frontend.
Our frontend app will be able to send requests to the backend app and the backend app will send responses to the frontend.
We'll need to run both servers at the same time. Then we can sent requests in our local environment.
To run the Rails API: rails s


# Angular project setup
We'll use Angular 17 for the frontend.
run ng version
[the notes recommend updating to version 17, but I read different opinions]
[I stuck with 16 for this example project]

ng new todo_list_frontend --no-strict

Then we'll generate a model:
ng g class models/todo

export class Todo {
	id: number = -1;
	body: string = '';
	is_completed: boolean = false;
}

Next we'll create a service for the todo list:
ng g s service/todo

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Todo } from '../models/todo';

@Injectable({
	providedIn: 'root',
})
export class TodoService {
	private url = 'http://localhost:3000/todos';

	constructor(private http: HttpClient) {}

	getTodos(): Observable<Todo[]> {
		return this.http.get<Todo[]>(this.url);
	}

	getTodoById(id: number): Observable<Todo> {
		return this.http.get<Todo>(`${this.url}/${id}`);
	}

	createTodo(todo: { body: string; is_completed: boolean }): Observable<Todo> {
		return this.http.post<Todo>(this.url, todo);
	}

	updateTodo(todo: Todo): Observable<Todo> {
		return this.http.put<Todo>(`${this.url}/${todo.id}`, todo);
	}

	deleteTodo(id: number): Observable<Todo> {
		return this.http.delete<Todo>(`${this.url}/${id}`);
	}
}

This service has defined methods to make requests to the Rails API.

Then we need to configure the app.config.ts file [version 17] or the app.module.ts file [version 16].
import { HttpClientModule } from '@angular/common/http';
and add it to the imports array

Then we'll create a component for the todo list:
ng g c todo-list --standalone

import { Component, OnInit } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { TodoService } from '../services/todo.service';
import { Todo } from '../models/todo';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-todo-list',
  standalone: true,
  imports: [FormsModule, CommonModule],
  templateUrl: './todo-list.component.html',
  styleUrl: './todo-list.component.scss'
})
export class TodoListComponent implements OnInit{
  todos: Todo[] = [];
  newTodoBody: string = ''; // Property to hold the new todo input value

  constructor(private todoService: TodoService) { }

  ngOnInit(): void {
    this.todoService.getTodos().subscribe(todos => this.todos = todos);
  }

  addTodo() {
    if (!this.newTodoBody.trim()) {
      // Prevent adding empty todos
      return;
    }

    const todo = {
      body: this.newTodoBody,
      is_completed: false
    };

    this.todoService.createTodo(todo).subscribe(newTodo => {
      this.todos.push(newTodo);
      this.newTodoBody = ''; // Reset the input field after adding
    });
  }

  updateTodo(todo: Todo) {
    this.todoService.updateTodo(todo).subscribe(updatedTodo => {
      const index = this.todos.findIndex(t => t.id === updatedTodo.id);
      this.todos[index] = updatedTodo;
    });
  }

  deleteTodo(id: number) {
    this.todoService.deleteTodo(id).subscribe({
      next: () => this.todos = this.todos.filter(todo => todo.id !== id),
      error: (err) => console.error(err)
    });
  }
}

In this component we've defined methods to add, update, and delete todos. 
We also get all the todos when the component is initialized.

Then we create a view for the todo list (in the HTML template):

<h1>Todo List</h1>

<!-- Todo form -->
<div class="todo-form">
	<input type="text" [(ngModel)]="newTodoBody" placeholder="Add a new todo..." />
	<button (click)="addTodo()">Add Todo</button>
</div>

<!-- Todo list -->
<ul>
	<li *ngFor="let todo of todos">
		<input type="checkbox" [(ngModel)]="todo.is_completed" (change)="updateTodo(todo)" />
		<span>{{ todo.body }}</span>
		<button (click)="deleteTodo(todo.id)">Delete</button>
	</li>
</ul>

Then we add some styles to the CSS [see project].

And we import the TodoListComponent into the app component.

We can start the app with: ng serve
And also make sure our API server is running: rails s

We'll get a CORS error
Access to XMLHttpRequest at 'http://localhost:3000/todos' from origin 'http://localhost:4200' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

CORS is a security feature that prevents requests from other domains from accessing the API.
We can fix this by adding the rack-cors gem to the Rails API.
gem 'rack-cors'
Then bundle install

In the config/initializers/cors.rb file:
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'
    resource '*', headers: :any, methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end

This allows requests from any origin to access the API. In a production environment we'll want to restrict this to only domains we trust. We replace * with the domains.


Then we'll need to restart the server. 