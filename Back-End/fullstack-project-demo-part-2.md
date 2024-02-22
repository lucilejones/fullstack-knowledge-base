Continuing from fullstack project part 1

# Incorporating Authentication

We'll create a model for User
rails generate model User username:string password_digest:string

rails db:migrate

We use the has_secure_password method to add authentication to the app.
This method expects a password_digest attribute in the database and virtual attributes password and password_confirmation for the model.

In the app/models/user.rb file:
class User < ApplicationRecord
    has_secure_password
    validates :username, presence: true
end

In order to use the has_secure_password method, we need to add the bcrypt gem to the Gemfile:
gem 'bcrypt'
bundle install

Then we'll create a Users Controller:
rails g controller users

In the app/controllers/users_controller.rb file:
class UsersController < ApplicationController
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
    params.permit(:username, :password, :password_confirmation)
  end
end

Then we'll add the gem 'jwt' and create a sessions controller:
gem 'jwt'
bundle install
rails g controller sessions create

In the app/controllers/sessions_controller.rb file:
class SessionsController < ApplicationController
  def create
    user = User.find_by(username: params[:username])
    if user&.authenticate(params[:password])
      token = jwt_encode(user_id: user.id)
      render json: { token: token }, status: :ok
    else
      render json: { error: "Unauthorized" }, status: :unauthorized
    end
  end

  private

  def jwt_encode(payload, exp = 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, Rails.application.secrets.secret_key_base)
  end
end

We add the following to our config/routes.rb file:
Rails.application.routes.draw do
  post '/login', to: 'sessions#create'
  resources :todos
  resources :users, only: [:create]
end

Then in the app/controllers/application_controller.rb file:
class ApplicationController < ActionController::API
  def authenticate_request
    header = request.headers['Authorization']
    header = header.split(' ').last if header
    begin
      decoded = JWT.decode(header, Rails.application.secrets.secret_key_base).first
      @current_user = User.find(decoded['user_id'])
    rescue JWT::ExpiredSignature
      render json: { error: 'Token has expired' }, status: :unauthorized
    rescue JWT::DecodeError
      render json: { errors: 'Unauthorized' }, status: :unauthorized
    end
  end
end

# Adding Auth to the front end
We're going to add login/logout

Login
We will need to sent a POST request to the server with the user's credentials. If the credentials are correct, the server will respond with a token. We'll store the token in local storage and use it to authenticate future requests.

ng g service services/authentication
In the service we'll define the following methods: login, setToken, getToken, isLggenIn, logout

In the services/authentication.service.ts file:
// authentication.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject } from 'rxjs';
import { Router } from '@angular/router';

@Injectable({
	providedIn: 'root',
})
export class AuthenticationService {
	private readonly tokenSubject = new BehaviorSubject<string | null>(null);

	constructor(private http: HttpClient, private router: Router) {}

	login(username: string, password: string) {
		return this.http.post<{ token: string }>('http://localhost:3000/login', {
			username,
			password,
		});
	}

	setToken(token: string) {
		localStorage.setItem('token', token);
		this.tokenSubject.next(token);
	}

	getToken() {
		return localStorage.getItem('token');
	}

	isLoggedIn() {
		return !!this.getToken();
	}

	logout() {
		localStorage.removeItem('token');
		this.tokenSubject.next(null);
		this.router.navigate(['/login']);
	}
}

Realistically we'd store the url in an environment variable (because there are different environments for develpment, production, testin, etc)

Next we'll create the login component:
ng g c login --standalone

Our loging component will need a form with two inputs, a method to handle the form submission and call the login method (from AuthenticationService), and a method to handle the response from the server and store the token in local storage.

In the app/login/login.component.ts file:
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { Router } from '@angular/router';
import { AuthenticationService } from '../services/authentication.service';

@Component({
	selector: 'app-login',
	standalone: true,
	imports: [FormsModule],
	templateUrl: './login.component.html',
	styleUrl: './login.component.scss',
})
export class LoginComponent {
	username: string = '';
	password: string = '';

	constructor(private authService: AuthenticationService, private router: Router) {}

	login() {
		this.authService.login(this.username, this.password).subscribe({
			next: (res: any) => {
				console.log('Logged in with token:', res.token);
				this.authService.setToken(res.token);
				this.router.navigate(['/todo-list']);
			},
			error: (error: any) => {
				console.error('Login error', error);
			},
		});
	}
}

[The above code is for version 17; I probalby tweaked it a bit for version 16.]

In the login.componenet.html file:
<form (ngSubmit)="login()">
	<input type="text" [(ngModel)]="username" name="username" placeholder="Username" required />
	<input type="password" [(ngModel)]="password" name="password" placeholder="Password" required />
	<button type="submit">Login</button>
</form>

In the app.routes.ts file:
import { Routes } from '@angular/router';

export const routes: Routes = [
	{ path: '', redirectTo: 'login', pathMatch: 'full' },
	{
		path: 'login',
		loadComponent: () => import('./login/login.component').then((m) => m.LoginComponent),
	},
];

Then we add router outlet to app.component.html to render componenents based on the route.
[This will be a little different in my todo_list_frontend that is version 16]

In the app.component.ts file:
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
	selector: 'app-root',
	standalone: true,
	templateUrl: './app.component.html',
	styleUrl: './app.component.scss',
	imports: [RouterOutlet],
})
export class AppComponent {}

In the app.component.html file:
<router-outlet />

In the login.component.css file (or scss):
form {
	max-width: 400px;
	margin: 2rem auto;
	padding: 2rem;
	background-color: #f9f9f9;
	border-radius: 8px;
	box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
	display: flex;
	flex-direction: column;
	gap: 1rem;
}

input[type='text'],
input[type='password'] {
	padding: 0.75rem;
	border: 1px solid #ccc;
	border-radius: 4px;
	font-size: 1rem;
	color: #333;
	background-color: white;
	transition: border-color 0.3s;
}

input[type='text']:focus,
input[type='password']:focus {
	border-color: #007bff; /* Light blue border for focus */
	outline: none;
}

button[type='submit'] {
	padding: 0.75rem;
	border: none;
	border-radius: 4px;
	background-color: #007bff; /* Light blue background */
	color: white;
	font-size: 1rem;
	font-weight: bold;
	cursor: pointer;
	transition: background-color 0.3s;
}

button[type='submit']:hover {
	background-color: #0056b3; /* Darker blue on hover */
}


Adding guards
Certain routes will require the user to be logged in - so we'll create a guard to protect those routes. 

ng generate guard auth
Then hit enter to choose CanActivate

In the src/app/auth.guard.ts file:
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthenticationService } from './services/authentication.service';

export const authGuard: CanActivateFn = (route, state) => {
	const authService = inject(AuthenticationService);
	const router = inject(Router);

	if (authService.isLoggedIn()) {
		return true;
	} else {
		router.navigate(['/login']);
		return false;
	}
};

Here we're using the CanActivateFn interface to create a guard that will check if the user is logged in. If the user is logged in, the guard will return true and the user will be allowed to access the route. If the user isn't logged in, the guard will redirect to the login page and return false.

We'll add the guard to our routes (specifically the todo list route).
In the app.routes.ts file:
import { Routes } from '@angular/router';
import { authGuard } from './auth.guard';

export const routes: Routes = [
	{ path: '', redirectTo: 'login', pathMatch: 'full' },
	{
		path: 'login',
		loadComponent: () => import('./login/login.component').then((m) => m.LoginComponent),
	},
	{
		path: 'todo-list',
		loadComponent: () => import('./todo-list/todo-list.component').then((m) => m.TodoListComponent),
		canActivate: [authGuard],
	},
];

Then we'll add a guard for routes that should only be accessible when a user is not logged in.

ng generate guard no-auth
[choose CanActivate and press enter]

In the src/app/no-auth.guard.ts file:
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthenticationService } from './services/authentication.service';

export const noAuthGuard: CanActivateFn = (route, state) => {
	const authService = inject(AuthenticationService);
	const router = inject(Router);

	if (authService.isLoggedIn()) {
		router.navigate(['/todo-list']);
		return false;
	} else {
		return true;
	}
};

Here, if the user is logged in, the guard will redirect to the list page and return false. If the user is not logged in, the guard will return true and the user will be allowed to access the route.

The the app.routes.ts file should look like this (with the canActivate: [noAuthGurad] added):
import { Routes } from '@angular/router';
import { authGuard } from './auth.guard';
import { noAuthGuard } from './no-auth.guard';

export const routes: Routes = [
	{ path: '', redirectTo: 'login', pathMatch: 'full' },
	{
		path: 'login',
		loadComponent: () => import('./login/login.component').then((m) => m.LoginComponent),
		canActivate: [noAuthGuard],
	},
	{
		path: 'todo-list',
		loadComponent: () => import('./todo-list/todo-list.component').then((m) => m.TodoListComponent),
		canActivate: [authGuard],
	},
];

Let's create a user in the Rails console and then we can test the guards.
rails c
User.create(username: 'test', password: 'password', password_confirmation: 'password')


# Adding logout
Becuase we're using a JWT, the server isn't responsible for keeping track or the user's session. We can delete the token to logout the user.

Create a navbar component:
ng g c navbar --standalone

In the navbar.component.ts file:
import { Component } from '@angular/core';
import { AuthenticationService } from '../services/authentication.service';
import { RouterLink } from '@angular/router';

@Component({
	selector: 'app-navbar',
	standalone: true,
	imports: [RouterLink],
	templateUrl: './navbar.component.html',
	styleUrl: './navbar.component.scss',
})
export class NavbarComponent {
	constructor(public authService: AuthenticationService) {}

	logout() {
		this.authService.logout();
	}
}

In the navbar.component.html file:
<nav class="stellar-navbar">
	<div class="brand-name">
		<a routerLink="/">Todo List</a>
	</div>
	<div class="nav-links">
		@if (authService.isLoggedIn()){
		<button (click)="logout()">Logout</button>
		}@else {
		<a routerLink="/login">Login</a>
		}
	</div>
</nav>


In the navbar.component.css file (or scss):
.stellar-navbar {
	display: flex;
	justify-content: space-between;
	align-items: center;
	background-color: #007bff; /* Primary color for the navbar */
	color: white;
	padding: 0.5rem 1rem;
}

.stellar-navbar .brand-name a {
	font-size: 1.5rem;
	font-weight: bold;
	color: white; /* Keeps the brand name white */
	text-decoration: none;
}

.stellar-navbar .nav-links {
	display: flex;
	align-items: center;
	gap: 1rem; /* Adds space between navigation items */
}

.stellar-navbar .nav-links a,
.stellar-navbar .nav-links button {
	background: none;
	border: 1px solid white; /* Border makes it stand out */
	border-radius: 4px;
	color: white;
	padding: 0.5rem 1rem;
	text-decoration: none;
	font-size: 1rem;
	cursor: pointer;
	transition: background-color 0.3s, color 0.3s;
}

.stellar-navbar .nav-links a:hover,
.stellar-navbar .nav-links button:hover {
	background-color: white;
	color: #007bff; /* Inverse the color scheme on hover */
}

/* Specific style for the logout button to differentiate it, if needed */
.stellar-navbar .nav-links button {
	background-color: transparent;
	color: white;
	border-color: white;
}

.stellar-navbar .nav-links button:hover {
	color: #007bff;
	background-color: white;
	border-color: #007bff;
}

In style.scss add the following to remove the default margin and padding from the body:
*,
html,
body {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'Roboto', sans-serif;
}

In the app component, we need to import the navbar and add it to the template:
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { NavbarComponent } from './navbar/navbar.component';

@Component({
	selector: 'app-root',
	standalone: true,
	templateUrl: './app.component.html',
	styleUrl: './app.component.scss',
	imports: [RouterOutlet, NavbarComponent],
})
export class AppComponent {}

<app-navbar /> 
<router-outlet />


# notes from class 2/22/2024

We'll set up a service and create a login method. We inject the HttpClient into the service. It'll be a post request to the sessions controller using the create action.

login(username: string, password:string) {
	return this.http.post(`${environment.apiUrl}/login`, {username, password})
}

We'll need the username and password from the user.
This returns an observable and we'll subscribe to it in our login component.
The response will send back a token that we can store.

setToken(token:string) {
	localStorage.setItem('token', token)
}

getToken() {
	return localStorage.getItem('token')
}

isLoggedIn() {
	return !!this.getToken();
}
This will return as a boolean.
string, number => return true
null, undefined => return false
This doesn't validate the token with the API.

logout() {
	localStorage.removeItem('token');
	this.router.navigate(['/login'])
}
In practice, we'll want to include an expiration date for the token.

For the input form, we'll import { ReactiveFormsModule } from '@angular/forms';


We can check in the localStorage has a token and if it does, we'll have the UI reflect that a user is logged in. If not, it should redirect to the login page.

@if (authService.isLoggenIn()) {
	<a (click)="authService.logout()">Logout</a>
}
@else {
	<a routerLink="/login">Login</a>
}

This will re-render every time. Instead, we should include a BehaviorSubject. 