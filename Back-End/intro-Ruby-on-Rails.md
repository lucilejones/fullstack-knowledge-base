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

The log foler contains the log files for our app. This is where we find info about what's happening in the app when it's running. We can find this while running the server or when we deploy the app to a server.

The public folder contains the static files for our app: images, JS, CSS

The storage folder contains files uploaded by users.

The test folders contains the tests for our app.

The tmp folder contains temporary files for our app.

The vendor folder contains third-party code that is not managed by Bundler.


# Interacting with the Database with Models


# Intro to Associations