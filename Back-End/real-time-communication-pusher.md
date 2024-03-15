# Real Time Communication with Pusher

Real-time web technologies allow users to receive information as soon as it is published.
Users can get instant updates and notifications; and don't need to refresh a page to see new content.
Traditional web communication
-request/response model
A client must initiate a new request in order to receive new or updated info.
Real-time web technologies
-publish/subscribe model
The server publishes information, and the client subscribes to receive updates instantly

WebSockets - the most common protocol for real-time communication
-provide a full-duplex communication channel over a single TCP connection
TCP is a reliable protocol that ensure messages are delivered in the order they were sent, and they they're not lost or duplicated.

Use cases:
-chat applications
users send and receive messages instantly

-live notifications
ex. social media platforms inform users about new posts, comments, like, etc without the need to refresh the page

-real-time updates
ex. financial trading platforms, sports scores website, news outlets, etc

-collaborative editing
tools like Google Docs allow multiple users to edit the document simultaneously

-IoT applications
Internet of Things devices often send data to servers in real-time; enable immediate monitoring and control of home appliances, security systems, industrial machinery, etc

Pros of using real-time web technologies:
Enhanced user experience
Increase interactivity
Improved efficiency
Reduced server load
Support for new application types

Cons of using real-time web technologies:
Increased complexity
Security concerns
Scalability challenges
Potential for overuse


# Understanding WebSockets

WebSockets are a protocol that provides a full-duplex communication channel over a single TCP connection. 
-allows for low-latency communication between client and server
-maintain a persistent connection between teh clinet and server, allowing the server to push updates to the client as soon as they're available.
-allows for instant data transfer in both directions without repeatedly establishing new connections or polling the server for updates.

Once a WebSocket connection is established, the client and server can freely send data back and forth until the connection is closed.
There's an initial handshake over the HTTP protocol, where the client requests and upgrade to WebSockets and the server responds acknowledging the upgrage.
The protocol used then switches from HTTP to WebSockets (ws:// or wss:// for secure connections)


# Action Cable vs Pusher

Action Cable - a real-time communication framework for Rails apps.
-provides a built-in WebSocket server and client_side JavaScript library
-integrated with the rest of the Rails stack

Action Cable has some limitations:
-requires additional configuration and setup, and may not be the best choice for apps that aren't built entirely with Rails
-may not be the most efficient for apps that require high scalability and performance
-designed to work with a single server

Pusher - a thrird-party service
-offers a simple and reliable way to add real-time features without the need to manage the infrastructure and complexity of WebSockets
-provides a set of client libraries and server-side SDKs
-integrates with a variety of technologies, including Ruby on Rails


# Getting started
https://pusher.com/

Will need a pusher account. We can use the sandbox plan.
Then we'll create a new app and obtain the needed credentials to integrate Pusher into our app.

Create a new Rails app:
rails new pusher_chat_app --api

# Integrating Pusher with Rails

Then we'll add the pusher gem to the Gemfile (to include in all environments):
gem 'pusher'

run bundle install

Then we get the necessary credentials from the Pusher dashboard.
We need to first create a new app.
We can use the default settings and click "Create my app".
If we're going to deploy the app, it may be helpful to check the "Create apps for multiple environments" to create separate apps for development, staging, and production.

Navigate to App Keys (in the dropdown arrow menu), and we'll be able to see:
app_id
key
secret
cluster

We need to run the follwowing to edit the credentials file:
EDITOR="code --wait" bin/rails credentials:edit --environment=development

By adding the --environment=development flog, we're specifying we want to edit the credentials for the development environment. We can also specify other envirionments, like test or production.
In the previous lesson we made the main credentials.yml.enc file for production.
We can separate the credentials for each environment.

Opens the credentials file.
Then we add the Pusher credentials to the file:
pusher:
  app_id: "453645"
  key: "dsfgfgh"
  secret: "sgdfjy"
  cluster: "us2"

Then we close the file to save it.

In the rails console, we can test this out, if we type
Rails.application.credentials.pusher[:app_id]

It should give us the app_id.

Next we'll create a pusher.rb file in the config/initializers directory:
require 'pusher'

Pusher.app_id = Rails.application.credentials.pusher[:app_id]
Pusher.key = Rails.application.credentials.pusher[:key]
Pusher.secret = Rails.application.credentials.pusher[:secret]
Pusher.cluster = Rails.application.credentials.pusher[:cluster]
Pusher.logger = Rails.logger

# Using Pusher to Broadcast Messages

We'll create a simple chat app for users to send and receive messages in real-time.
Pusher will broadcast messages to all connected clients whenever a new message is sent.
We won't use authentication for this example, but in a real-world app we'd need to implement authentication and authorization.

First, we'll create a new model Message:
rails g model Message content:text

rails db:migrate

Then we'll create a controller MessagesController to handle the creation and retrieval of messages:
rails g controller Messages

In the app/controllers/messages_controller.rb file:
class MessagesController < ApplicationController
  def index
    messages = Message.all
    render json: messages
  end

  def create
    message = Message.new(message_params)

    if message.save
      Pusher.trigger('chat', 'new_message', message.as_json)
      render json: message, status: :created
    else
      render json: message.errors, status: :unprocessable_entity
    end
  end

  private

  def message_params
    params.permit(:content)
  end
end

When as user creates a new message, it'll fire off a new event called new_message to the chat channel. This event will be broadcast to all connected clients.

Then we'll define the routes for the messages controller.

In the config/roues.rb file:
Rails.application.routes.draw do
  resources :messages, only: [:index, :create]
end

# Configuring CORS
We'll need to configure CORS to allow the front-end app to make requests to the Rails API.

Add the rack-cors gem to the Gemfile:
gem 'rack-cors'

run bundle install

We'll add the following in the config/initializers/cors.rb file:
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins "*"

    resource "*",
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end

# Front End Setup

Create a new Angular app
ng new fe-pusher-chat-app

We'll need to install the Pusher JavaScript library
-enables real-time communication with the Pusher service
-provides client-side interface for subscribing to channels and receiving real-time updates
https://github.com/pusher/pusher-js

npm install pusher-js

Then we'll create a message.service.ts to handle the communication with the Rails API and the Pusher channel:
ng generate service message

In the src/app/message.service.ts file:
import { Injectable } from '@angular/core';
import Pusher from 'pusher-js';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})

export class MessageService {
  private pusher: any;
  private channel: any;

  constructor(private http: HttpClient) {
    this.pusher = new Pusher('6ed32465d0c1fff140be76a', {
      cluster: 'us2'
    });
    this.channel = this.pusher.subscribe('chat');
  }

  getMessages(): Observable<any> {
    return this.http.get('http://localhost:3000/messages');
  }

  sendMessage(message: string): Observable<any> {
    return this.http.post('http://localhost:3000/messages', { content: message });
  }

  subscribeToNewMessages(callback: (message: any) => void): void {
    this.channel.bind('new_message', callback);
  }

  unsubscribeFromNewMessages(): void {
    this.channel.unbind('new_message');
  }

}

We import the Pusher library to use it.
In the constructor, we create a new instance of the Pusher client and subscribe to the chat channel.
We user the key and cluster credentials from the Pusher dashboard to initialize the Pusher client.
The channel is the same channel we're broadcasting the messages to from the Rails API.

We define two methods to get and send messages.

We need to also configure the app.config.ts file:
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes), provideHttpClient()]
};

We're importing provideHttpClient; this will allow us to use the HttpClient module in our service.

# Adding Real-Time features

Next we'll create a simple chat interface.

In the src/app/app.component.html file:
<div style="text-align:center">
  <h1>
    Welcome to Pusher Chat App
  </h1>
</div>
<div>
  @for (message of messages; track message.id) {
    <p>{{ message.content }}</p>
  }
</div>
<div>
  <input type="text" [(ngModel)]="newMessage" />
  <button (click)="sendMessage()">Send</button>
</div>

In the src/app/app.component.ts file:
import { Component, OnInit } from '@angular/core';
import { MessageService } from './message.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})

export class AppComponent implements OnInit {
  messages: any[] = [];
  newMessage: string = '';

  constructor(private messageService: MessageService) {}

  ngOnInit(): void {
    this.messageService.getMessages().subscribe((messages: any) => {
      this.messages = messages;
    });

    this.messageService.subscribeToNewMessages((message: any) => {
      this.messages.push(message);
    });
  }

  sendMessage(): void {
    this.messageService.sendMessage(this.newMessage).subscribe((message: any) => {
      this.messages.push(message);
      this.newMessage = '';
    });
  }
}

One problem:
When the user sends a message, the message will be added to the messages array twice; once when the user sends a message and once when the message is received from the Pusher channel.
To fix this, we can check to see if the message already exists in the array before pushing it:
sendMessage(): void {
    this.messageService.sendMessage(this.newMessage).subscribe((message: any) => {
      if (!this.messages.find((m: any) => m.id === message.id)) {
        this.messages.push(message);
      }
      this.newMessage = '';
    });
  }


Needed to import the FormsModule into the component and make it standalone: true,
...
import { FormsModule } from '@angular/forms';

@Component({
...
  standalone: true,
  imports: [FormsModule],
...
})


# notes from class 3/14/2024
rails g model Like user:references likeable:references{polymorphic}

The Like belongs to a user and can be for any kind of record (blog post, photo, comment, etc)

In the user.rb:
has_many :likes

In the blog.rb:
has_many :likes, as: :likeable

the like.rb file:
belongs_to :user
belongs_to :likeable, polymorphic: true

def like
  #@current_user
  #blog_id
  blog = Blog.find(params[:blog_id])
  #allow user to like blog
  like = blog.likes.new(user_id: @current_user.id)
  blog_creator = blog.user

  if like.save
    #add Pusher.trigger code here
    Pusher.trigger(blog_creator.id, 'like', {
      blog_id: blog.id,
      notification: "#{@current_user.username} has liked #{blog.title}"
    })
    head :ok
  else
    render json: nil, status: :unprocessable_entity
  end
end

def unlike
  blog = Blog.find(params[:blog_id])
  like = blog.likes.find_by(user_id: @current_user.id)

  if like.destroy
    head :ok
  else
    render json: nil, status: :unprocessable_entity
  end
end

Then we need to update the blueprint for blog in order to include the likes info.
field :liked do |blog, options|
  blog.likes.where(user: options[:current_user]).exists?
end

To refactor that, we can define a method in the blog model and then include that in the blueprint.

Adding Angular Material:
ng add @angular/material

And we'll need to import the MatIconModule in the TS file for the blog-list.component

On the frontend, we'll update the Blog model to include liked: boolean = false;

Then we use an EventEmitter. 

With Pusher we need to user the pusher-js library.

npm install pusher-js

We need to set up the development environment:
pusher: {
  key: 'asdfs',
  cluster: 'us2'
}

We can put the same info in the production environment, but in a real-world scenerio it probably makes sense to use separate info for each environment.

Then we'll create a notificaiton service.
We'll have the user subscribe to a channel when they log in.

Following the Puhser docs to set up a subscription to a user's channel when they log in, and listen to specific events.

listen(userId: number) {
  this.pusher = new Pusher(environment.pusher.key, {
    cluster: environment.pusher.cluster
  });

  this.channel = this.pusher.subscribe(userId.toString());

  this.channel.bind('like', (data: any) => {
    console.log('received data', data)
    ...
  })
}

In the backend, we set up a route '/web' to get data when the app loads, when a user is logged in. So we can get the current_user.id.

And we'll set up a user.service.ts
