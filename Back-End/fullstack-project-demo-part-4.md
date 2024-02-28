[there was no part 3]

# Signing up a new user
We'll need to create a sign-up form.

First, we'll update the authentication service to include a method to signup a new user.
In the authentication.service.ts file:
	signUp(user: any) {
		return this.http.post('http://localhost:3000/users', user);
	}

Then we'll make a signup component:
ng g c signup --standalone

In the signup.component.ts file:
import { Component } from '@angular/core';
import { AuthenticationService } from '../services/authentication.service';
import { Router } from '@angular/router';
import { FormsModule } from '@angular/forms';

@Component({
	selector: 'app-signup',
	standalone: true,
	imports: [FormsModule],
	templateUrl: './signup.component.html',
	styleUrl: './signup.component.scss',
})
export class SignupComponent {
	user = {
		username: '',
		password: '',
		password_confirmation: '',
	};

	constructor(private authService: AuthenticationService, private router: Router) {}

	onSubmit() {
		if (this.user.password === this.user.password_confirmation) {
			this.authService.signUp(this.user).subscribe({
				next: (res: any) => {
					console.log('Sign up successful', res);
					// Redirect to login or another page
					this.router.navigate(['/login']);
				},
				error: (error: any) => {
					console.error('Sign up failed', error);
					// Handle error (e.g., show error message)
				},
			});
		} else {
			console.error('Passwords do not match');
			// Handle password mismatch (e.g., show error message)
		}
	}
}

In the signup.component.html file:
<div class="signup-container">
	<h2>Sign Up</h2>
	<form (ngSubmit)="onSubmit()" #signupForm="ngForm">
		<div class="form-group">
			<label for="username">Username</label>
			<input type="text" id="username" [(ngModel)]="user.username" name="username" required />
		</div>
		<div class="form-group">
			<label for="password">Password</label>
			<input type="password" id="password" [(ngModel)]="user.password" name="password" required />
		</div>
		<div class="form-group">
			<label for="password_confirmation">Confirm Password</label>
			<input
				type="password"
				id="password_confirmation"
				[(ngModel)]="user.password_confirmation"
				name="password_confirmation"
				required
			/>
		</div>
		<button type="submit" class="btn" [disabled]="!signupForm.valid">Sign Up</button>
	</form>
</div>

In the signup.component.css file:
.signup-container {
	max-width: 400px;
	margin: 2rem auto;
	padding: 2rem;
	background-color: #f7f7f7;
	border-radius: 8px;
	box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.form-group {
	margin-bottom: 1rem;
}

.form-group label {
	display: block;
	margin-bottom: 0.5rem;
	font-weight: bold;
}

.form-group input {
	width: 100%;
	padding: 0.75rem;
	border: 1px solid #ccc;
	border-radius: 4px;
	font-size: 1rem;
	color: #333;
}

.btn {
	display: inline-block;
	background-color: #007bff;
	color: white;
	padding: 0.75rem 1.5rem;
	border: none;
	border-radius: 4px;
	cursor: pointer;
	font-size: 1rem;
	text-align: center;
	width: 100%;
	transition: background-color 0.3s ease;
}

.btn:hover {
	background-color: #0056b3;
}

/* Additional styles for error messages or validation feedback */
.error-message {
	color: #ff3860;
	margin-top: 0.5rem;
}

/* Style adjustments for better responsiveness on smaller screens */
@media (max-width: 576px) {
	.signup-container {
		padding: 1rem;
		margin: 1rem;
	}
}

Then we need to add the route for the signup page.
(In app.routes.ts for version 17; in the app.module.ts for version 16)
	{
		path: 'signup',
		loadComponent: () => import('./signup/signup.component').then((m) => m.SignupComponent),
		canActivate: [noAuthGuard],
	},

And we need to make sure to add the signup link to the navbar:
<nav class="stellar-navbar">
	<div class="brand-name">
		<a routerLink="/">Todo List</a>
	</div>
	<div class="nav-links">
		@if (authService.isLoggedIn()){
		<button (click)="logout()">Logout</button>
		}@else {
		<a routerLink="/login">Login</a>
		<a routerLink="/signup">Sign Up</a>
		}
	</div>
</nav>


# Grabbing the user's todos

We'll want to include the user's token for certain requests. 

In the backend we'll add an user_id column to the todos table:
rails g migration add_user_id_to_todos user:references

rails db:migrate

If there are existing todos in the database, we'll get an error. Becuase the user_id column is not nullable.
We need to reset the database and then migrate.

rails db:reset
(This will drop the database, create the database, load the schema, and then seed the database)

rails db:migrate

Then if we check the schema file we'll see the user_id added to the todos table.

In the app/models/todo.rb file:
belongs_to :user

In the app/models/user.rb file:
has_many :todos

Then we need to add a route to fetch the user's todos.
In the config/routes.rb:
get '/my_todos', to: 'todos#my_todos'

In the todos_controller, we'll add the my_todos method:
...
def my_todos
    todos = @current_user.todos

    render json: TodoBlueprint.render(todos, view: :normal), status: :ok
end

We also need to add the authenticate_request method (with before_action)


Then we'll update the frontend.
We'll add a method to the todo service to fetch the user's todos.
In the app/services/todo.service.ts file:
getMyTodos(): Obervable<Todo[]> {
    return this.http.get<Todo[]>('http://localhost:3000/my_todos');
}

Then in the todo-list component, we'll replace the getTodos method (in the mgOnInit) with getMyTodos.
ngOnInit(): void {
    this.todoService.getMyTodos().subscribe(todos => this.todos = todos);
}

Now we need to include the user's token in the request headers.
We'll create an interceptor:
ng generate interceptor auth-token

In the auth-token.interceptor.ts file (this is for version 17):
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthenticationService } from './services/authentication.service';

export const authTokenInterceptor: HttpInterceptorFn = (req, next) => {
	const authService = inject(AuthenticationService);
	const authToken = authService.getToken();

	const authReq = authToken
		? req.clone({
				headers: req.headers.set('Authorization', `Bearer ${authToken}`),
		  })
		: req;
	return next(authReq);
};

Then we need to add the interceptor to the app.config.ts file:
(This is for version 17)
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authTokenInterceptor } from './auth-token.interceptor';

export const appConfig: ApplicationConfig = {
	providers: [provideRouter(routes), provideHttpClient(withInterceptors([authTokenInterceptor]))],
};


Create a Todo
Because we changed our todos controller to account for the authentication of a request, we need to update our create method.
def create
    todo = @current_user.todos.new(todo_params)

    if todo.save
      render json: TodoBlueprint.render(todo, view: :normal), status: :created
    else
      render json: todo.errors, status: :unprocessable_entity
    end
  end

