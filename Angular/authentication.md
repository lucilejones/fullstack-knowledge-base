# intro
Lots of apps require users to signup and login (at least to use certain features)

# How authentication works
We have a client and we have a server.
When a user enters their credentials, that auth data is sent to the server for validation. We can't do that validation in the browser because all the JS code in the browser is exposed to our users. 
Working with Angular and single page applications, that means we decouple the frontend from the backend. For the different "pages" that we visit, that all gets handled by Angular and its router.
For requests we're reaching out to a server, as a RESTful API, which does not use a session because RESTful APIs are stateless. (The same is true for a GraphQL API.)
Our server will be an API, it is not a server that renders the HTML pages, so sessions can't be used. The server and client are decoupled. They communicate through the HttpClient, but there is no connection.

Instead of sessions, we'll use a different approach. The server will validate the user email and password and send back a token - a JSON web token, typically. That is an encoded string which contains metadata. It's encoded, not encrypted. Which means it could be unpacked and read by the client. The token that is generated on the server with a certain algorithm and a certain secret which only the server knows and only the server can validate incoming tokens. 
The idea is that the client, our Angular app, stores that token (like in local Storage), and then attaches the token to any request sent after that needs to be authenticated. 
The server is able to validate the token because it created the token with a certain algorithm and with a private key that is only known to the server.

(Max is adding all these to the recipes app)
# adding the auth page
We need to add an interface where users can enter their credentials
This would be accessible in an unauthenticated state.

We'll make a folder in the app called auth.
In there we need an auth.component.ts and an auth.component.html

In our TS file, we export a class AuthComponent {}
We need to import the @Component decorator and define a selector and a templateUrl.
We also need to go to the app module and add this component to the declarations array (and import it).

Then in the template we'll create the form for the signup and login.

In the app-routing we need to register this component and its path.
{ path: 'auth', component: AuthComponent }

We'll add logic to make sure we're automatically redirected to the auth page if we're not logged in.

For the useability in this project, we can create a link in the header that goes to the auth page:
<li routerLinkActive="active"><a routerLink="/auth">Authenticate</a></li>

# Switching between auth modes
We'll need to manage the currently active mode.
In the auth component we'll add a property isLoginMode. The idea here is to store whether the user is currently logged in or in sign up mode, and we adjust the user interface. Also, what we do in the form gets submitted dynamically based on that property.
Then we add a method, onSwitchMode to change that property.
This allows us to dynamically change what's on the user interface when an event is triggered by the user using that helper property.
    onSwitchMode() {
        this.isLoginMode = !this.isLoginMode;
    }

Now we have to connect onSwitchMode() and this isLoginMode property in our template.

We want to switch the mode when we click the button.
So we add a click to the button, we call onSwitchMode()

Then we need to change the interface. The form inputs shouldn't change, but the text on the buttons should.
<button class="btn btn-primary" type="submit">{{ isLoginMode ? 'Login' : 'Signup' }}</button>
(Also, this button should submit the form, so we should give it type submit. The other button should have type button so it doesn't submit the form.)
            <div>
                <button class="btn btn-primary" type="submit">
                    {{ isLoginMode ? 'Login' : 'Sign Up'}}
                </button> |
                <button class="btn btn-primary" (click)="onSwitchMode()" type="button">
                    Switch to {{ isLoginMode ? 'Sign Up' : 'Login' }}
                </button>
            </div>

# handling form input
The input for email with take the attribute ngModel and we give it a name: email. We also want to validate it, so we'll add required and email.
                <label for="email">E-mail</label>
                <input
                    type="email"
                    id="email"
                    class="form-control"
                    ngModel
                    name="email"
                    required
                    email
                />

For the password we'll also add ngModel, a name, requried, and a minimum length.
                <label for="password">Password</label>
                <input
                    type="password"
                    id="password"
                    class="form-control"
                    ngModel
                    name="password"
                    required
                    minlength="6"
                />

In order to disable the button when the form isn't validated, we'll on the form get access to the form object Angular creates. We create a local reference, here we'll call it authForm and set it equal to ngForm.
<form #authForm="ngForm">

Then we disable the button if the authForm isn't valid:
<button class="btn btn-primary" type="submit" [disabled]="!authForm.valid">

When we submit the form, we'll add ngSubmit to trigger the onSubmit method and forward the authForm.
<form #authForm="ngForm" (ngSubmit)="onSubmit(authForm)">

Then in the TS file we add onSubmit()
We'll get the form which is of type NgForm and we need to import that from @angular/forms
    onSubmit(form: NgForm) {
        console.log(form.value);
        form.reset();
    }

# pretparing the backend
[in order to send successful authenticted requests for this project, we need to make sure we have recipes stored in the backend database. We'll want to add some before we turn on protection as shown in the last lecture]

We can add authentication to any Angular app. We don't need to use firebase. But whatever backend we use needs to offer endpoints we can use to create new users and login users to get a token. 

In the firebase interface, we go to the Authentication tab and set up a sign-in method. Before that we need to change the rules. What Max has on his:
{
    "rules": {
        ".read": "auth != null",
        ".write": "auth != null"
    }
}
This tells Firebase that only authenticated users can read and write data.

So we click on Authentication, then set up sign-in method, and choose email/password. Then we enable the first option (allow users to signup using their email...), then click save.
Now we can use the built-in Firebase authentication where we can send requests to certain endpoints offered by Firebase to create users and login users, etc.
We'll also be able to see a list of users (under Authentication in the users tab).

# preparing the signup request
We can google firebase auth rest api to find an article in the official docs for the endpoints made available by Firebase. It's a different REST api than we used before - it's not our datbase REST api. Instead, it's a dedicated authentication API offered by Firebase. We need to methods: signing up users and logging in users.

There will be a URL given where we need to send the request. They look a little different now than when Maz did the video but should work the same way. It'll be similar to this:
https://gooleapis.com/identitytoolkit/v3/relyingparty/signupNewUser?key=[API_KEY]

We'll need to send the email, the password, and the returnSecoreToken - a boolean that should always be true.
We'll get back a response with the email address, the id of the user (Firebase automatically creates a unique id for each user), and the idToken. We also get an expiresIn property because these tokens from Firebase expire in an hour. We'd need to refresh the token or login again.

In the auth folder, we'll add a service to deal with requests: auth.service.ts
This service will be responsible for signing up users, logging in users, and managing tokens.

This service will receive the @Injectable decorator (imported from @angular/core), and we'll pass the object the says providedIn: 'root'

We can start by adding a signup method. This should send a request to the signup URL.
So we'll need the Angular HttpClient, which needs to be injected (and also imported). So we'll add the constructor and create a private property called http.

In the signup method we can then use the HttpClient to send a POST request.
We'll also need to inclue our API_KEY at the end of the url.
In the Firebase console, clicking on the gear icon, then under project settings, it's the Web API key.

Since it's a post request we also need to attach some data. We'll attach a JS object that will hold the 3 properties we need to send (email, password, token boolean).
    signup(email: string, password: string) {
        this.http.post ('url', {
            email: email,
            password: password,
            returnSecureToken: true
        });
    }

This will now send an http request to the API server.
But we have to subscribe to the observable and then call the signup method. 
We'll return the prepared observable so we can subscribe in the auth component, which is where we'll be interested in getting a response. 
    signup(email: string, password: string) {
        return this.http.post ('url', {
            email: email,
            password: password,
            returnSecureToken: true
        });
    }

We can also define the format of the data we'll get back. We'll get back the six fields shown in the Firebase documentation.

We create a new interface in the authService called AuthResponseData to define what the response data will look like.
interface AuthResponseData {
    kind: string;
    idToken: string;
    email: string;
    refreshToken: string;
    expiresIn: string;
    localId: string;
}

We define the interface here to define the types of data we're working with. All the http verbs are generic methods so we can hint the kind of data we'll get back. After the post, we can add the angle brackets and pass AuthresponseData.
        return this.http.post<AuthResponseData>('url', {
            email: email,
            password: password,
            returnSecureToken: true
        });

# sending the signup request
In the onSubmit method (in the auth component ts file) instead of logging to form values, we'll extract the email and password
        const email = form.value.email;
        const password = form.value.password;

We can also add an if check to just return if the form is not valid.
    onSubmit(form: NgForm) {
        // console.log(form.value);
        if (!form.valid) {
            return;
        }
        const email = form.value.email;
        const password = form.value.password;
        form.reset();
    }

Then we need to inject the authService into the auth component so we can call the signup method.
We add the constructor and store the authService in a private property (and import it from the auth.service file path)

Then we can call this.authService.signup and forward the email and password.
this.authService.signup(email, password);

We'd previously set up the signup method to return the prepared observable, which is ready to send the request, so we need to subscribe to the return value of signup, which is that observable.
this.authService.signup(email, password).subscribe();

In that response, we get our responseData, which we can just console.log for now. We might also get an error, and we can log that.
        this.authService.signup(email, password).subscribe(
            resData => {
                console.log(resData);
            },
            error => {
                console.log(error);
            }
        );

We'll need to change this to check if we're in login mode. (We don't have any logic to put in the block for when login mode is true, because we haven't written the login method yet.)

    onSubmit(form: NgForm) {
        // console.log(form.value);
        if (!form.valid) {
            return;
        }
        const email = form.value.email;
        const password = form.value.password;

        if (this.isLoginMode) {
            // ...
        } else {
            this.authService.signup(email, password).subscribe(
                resData => {
                    console.log(resData);
                },
                error => {
                    console.log(error);
                }
            );
        }

        form.reset();
    }

So then we can signup with an email and password. If it's successful, we'll get back that response object and we can see the user in the list of users in the Firebase interface.
Then if we try to signup with the same email again, we'll get a 400 error and a message from Firebase "EMAIL_EXISTS"

# Adding a loading spinner and Error handling logic
google css loading spinners
Go to loading.io/css

We can make a loading-spinner folder in the shared folder with a loading-spinner.component.css and a loading-spinner.component.ts

import { Component } from '@angular/core';

@Component({
    selector: 'app-loading-spinner',
    template: '<html>from the spinner we copied</html>',
    styleUrls: ['./loading-spinner.component.css']
})
export class LoadingSpinnerComponent {}

We need to declare that component in the app module and import it.

Then in the auth component, we want to hide the form if we're loading.
In the TS file we add a property isLoading and set it to false.
In onSubmit, right before the request gets sent, we can set this.isLoading to true. Then we set it back to false when we're done (in both the success case and the error case).

Then the isLoading property can be used in the template to hide the form. Just with an *ngIf.
<form #authForm="ngForm" (ngSubmit)="onSubmit(authForm)" *ngIf="!isLoading">

If we are loading, we want to show the spinner. 
<div *ngIf="isLoading">
    <app-loading-spinner></app-loading-spinner>
</div>

If we want to show an error message - an alert - if there was a problem with logging in. 
In the auth component TS file we'll add a property called error. Initially it will be null. (It will be of type string.)
error: string = null;

In the error case in the onSubmit function, instead of logging the error, we'll set the error equal to a message.
this.error = 'An eror occurred.';

In the template we can add a div to show when there is an error.

<div class="alert alert-danger" *ngIf="error">
    <p>{{ error }}</p>
</div>

# improving error handling
Firebas has common error codes. We can check for these and provide a more helpful error message.
In the auth component, we can update the error case (rename the parameter to be the errorRes and add a switch):
                errorRes => {
                    console.log(errorRes);
                    switch (errorRes.error.error.message) {
                        case 'EMAIL_EXISTS':
                            this.error = 'This email already exists';
                    }
                    this.error = 'An error occured.';
                    this.isLoading = false;
                }

We could do the above in the component; however, handling the error in the component is not necessarily the best way to do it. It moves too much logic into the component (rather than updating the UI).
Instead we can use an rxjs operator that will allow us to handle errors in the auth service (in the observable chain we're setting up).

So in the auth service, in the signup function, we can .pipe() on the observable and add rxjs operators.
We'll use the catchError operator (and import it from rxjs/operators). We'll also need to import throwError from rxjs.
throwError will create a new observable that wraps an error.
So we can move the switch from the component to the service.
Though we need to do it a bit differently, adding a new variable, errorMessage.
    signup(email: string, password: string) {
        return this.http.post<AuthResponseData>('url',
            {
                email: email,
                password: password,
                returnSecureToken: true
            }
        ).pipe(catchError(errorRes => {
            let errorMessage = 'An unknownn error occurred.';
            switch (errorRes.error.error.message) {
                case 'EMAIL_EXISTS':
                    errorMessage = 'This email already exists';
            }
        }));
    }

The switch statement could fail if the error we get doesn't have that same format, which migh tbe the case if we get a network error. So we can use an if to check whether it has an error key with a nested error key. If it doesn't, we'll return throwError and wrap the error message so we'll throw an observable that wraps the error message.
            if (!errorRes.error || !errorRes.error.error) {
                return throwError(errorMessage);
            }

We can add more cases to our switch statement later. But after, we want to return our throwError with the errorMessage.

Then in the component in the error case we just get the error message because we subscribe to an observable here which in the error case will be an observable that only contains a message.
So we can console.log it, and also set this.error = errorMessage.

# sending login requests
In the auth service we'll create a new login method, which also takes an email and password. 
We still have to send a request, and we need to check the Firebase docs for the URL and that it's a post request. (We'll also need the API key)
    login(email: string, password: string) {
        this.http.post('url',
            {
                email: email,
                password: password,
                returnSecureToken: true
            }
        );
    }

We get back almost the same response as in the login, but we'll also get an additional field, registered which is a boolean.
So in the auth service, in our AuthResponseData interface, we can add an optional registerd property.
interface AuthResponseData {
    kind: string;
    idToken: string;
    email: string;
    refreshToken: string;
    expiresIn: string;
    localId: string;
    registered?: boolean;
}

So we can add the angle brackets after post and inform TS about the type of the response.
this.http.post<AuthResponseData>('url',...)

And just like the signup, we want to return the observable so we prepare the observable here, but can subscribe in the auth component.
return this.http.post<AuthResponseData>('url',...)

Then in the auth component, in the isLoginMode we can reach out to the this.authService.login method and also forward email and password.
Then we can grab the same subscribe block from the signup method (because we'll still want to control the loading state and the error).
.subscribe(
                resData => {
                    console.log(resData);
                    this.isLoading = false;
                },
                errorMessage => {
                    console.log(errorMessage);
                    this.error = errorMessage;
                    this.isLoading = false;
                }
            );

Since it's the exact same code, we can write it up differently and not repeat ourselves.

We can create a new variable (within the onSubmit) called authObs (for auth observable), which will be of type Observable, that we need to import from rxjs.
That observable is a generic type, so we need to identify what kind of data we'll yield, and that will be out AuthResponseData. So we need to export that interface from the auth service to we can import it into the auth component. 
The idea is that we change the observable that authObs variable holds in the two branches of our if check (for the login vs signup mode). Then after the if check, one of the two observables will be stored there, so we can subscribe on that observable after the if block. And we only change what's in the observable inside the if block.
So for logging in, we set authObs = this.authService.login(email, password)
        if (this.isLoginMode) {
            authObs = this.authService.login(email, password);
        } else {
            authObs = this.authService.signup(email, password);
        }

        authObs.subscribe(
            resData => {
                console.log(resData);
                this.isLoading = false;
            },
            errorMessage => {
                console.log(errorMessage);
                this.error = errorMessage;
                this.isLoading = false;
            }
        );

Then when we login a user, we get the response data back in the console. But we do need to set up error handling for when a user tries to login with email and password that are not saved as a user.

# login error handling
It'd be nice to share the error handling logic between both the signup and the login observables.
We can create a new private method in the auth service that we call handleError and it receives and errorRes which is of type HttpErrorResponse (imported from @angular/common/http). 
    private handleError(errorRes: HttpErrorResponse) {}

In the code block we can take what was previously in the catchError method in the .pipe() on the signup method.
    private handleError(errorRes: HttpErrorResponse) {
        let errorMessage = 'An unknownn error occurred.';
        if (!errorRes.error || !errorRes.error.error) {
            return throwError(errorMessage);
        }
        switch (errorRes.error.error.message) {
            case 'EMAIL_EXISTS':
            errorMessage = 'This email already exists';
        }
        return throwError(errorMessage);
    }

Then in catchError, we pass this.handleError.
.pipe(catchError(this.handleError));

Then we can add that same pipe to the login observable.
We'd want to add to the switch case the EMAIL_NOT_FOUND and/or INVALID_PASSWORD errors (and possibly give the same error message to not give away whether it's the email or password that's incorrect).

# creating and storing the user data
In the auth folder, we'll create a user.model.ts file - we want to start storing info about a user, whether the user is authenticated, etc.
export class User {
    constructor(
        public email: string,
        public id: string,
        private _token: string,
        private _tokenExpirationDate: Date
    ) {}
}

We use a constructor so we can create a user with the new keyword; we add an accessor in front of the argumnet name which will also be used as the property name, in order to store arguments of the constructor in properties of the class. 

The _token and _tokenExpirationDate (this one will be of type Date), need to be private so that when we're trying to access the token, we should be required to do that in a way that will automatically check the validity. This can be acheived by adding a getter with the get keyword. A getter looks like a function, but we actually access it like a property. We'll be able to do something like, user.token
We can do this because it's a special type of property. We can write code that runs we when try to access this property. A getter also means that the user can't overwrite this. We can't use user.token = somethingnew, because it's a getter, not a setter. There are setters, but we don't need them here. 
We set a token and create a new user object. We will always create a new user object when a user logs in. 

In the getter we return this._token
But before we do that we can add certain checks.
We'll want to check if this._tokenExpirationDate does not exist or if the current date/timestamp is great than this token expiration date.
    get token() {
        if (!this._tokenExpirationDate || new Date() > this._tokenExpirationDate) {
            return null;
        }
        return this._token;
    }

With that created, in the auth service we want to store the authenticated user. We'll store it as a subject. 
Before the constructor, we'll add user = new Subject() (which needs to be imported from rxjs), and it will be a generic type where we store the User (which needs to be imported from our User model file).
user = new Subject<User>();

So then we emit - we next() - a new user whenever one logs in (or when we logout, when we clear the user).

To start, in the signup method in the auth service, we know that we get back the AuthResponseData which contains the email and the token we need for the user and the expiry date. 
So we add a new rxjs operator to the .pipe() after catchError, tap.
The tap operator needs to be imported from rxjs/operators, and it's an operator that allows us to perform an action without changing the response. It steps into the observable chain but doesn't stop, block, or change it. It just runs some code with the data we get back from the observable.

So there we get the response data and we want to create our new user.
We'll user the User model and forward the email, the id (the resData.localId - generated by Firebase), the token (our idToken), and the expiry date (which should be a Date object). The Date is not part of our response. It's something we need to generate. We can add a new constant and create a new JS object based on the expiresIn time we get back fro Firebase. 
expiresIn is a string (from the Firebase docs), but in the end it holds a number - the number of seconds til the id token expires.

So we have to get the current date and on that we can call getTime() - that's the current timestamp in milliseconds. Then we add the resData.expiresIn (which we have to convert to a number). We also have to do * 1000, because expiresIn is in seconds, but the timestamp is in milliseconds. 
By wrapping it in new Date it becomes a Date object instead of being in milliseconds.
Then we add that to the new User object. We pass it as a fourth argument to the user constructor.
.pipe(catchError(this.handleError), tap(resData => {
            const expirationDate = new Date(new Date().getTime() + +resData.expiresIn * 1000);
            const user = new User(resData.email, resData.localId, resData.idToken, expirationDate);
        }));

This constructs the user with the data we get back from the back-end and with our expiry date.
Now we can use the Subject to next() that user. 
this.user.next(user);
To set this - or emit this - as our now currently logged in user. 

We need the exact same logic for logging users in, so we can use the same approach.

We add a new private method, handleAuthentication(), and in that method we expect to get email, token, and expresIn as a number.
We can take the code we just wrote in tap and move tha to the handleAuthentication function. 
expiresIn is no longer coming from the resData and doesn't need to be converted to a number. 
Then we'll also pass in the userId (instead of localId) and the token. 
    private handleAuthentication(email: string, userId: string, token: string, expiresIn: number) {
        const expirationDate = new Date(new Date().getTime() + expiresIn * 1000);
        const user = new User(
            email, 
            userId, 
            token, 
            expirationDate
        );
        this.user.next(user);
    }

Then in tap, when we get the resData we just need to call this.handleAuthentication(); and forward the resData.
            tap(resData => {
                this.handleAuthentication(
                    resData.email,
                    resData.localId,
                    resData.idToken,
                    +resData.expiresIn);
            })

Then we can copy the tap operator and move it into the pipe() for our login route as well.

Then next step will be to redirect the user in the UI, update the UI, and show a logout button, etc.

# reflecting the auth state in the UI
First we'll forward to user to a different route once they're logged in. 
There are two different places we can do that - in the handleAuthentication method (in the auth service), or we can do it in the component in the subscribe (authObs.subscribe), in the success case only.
Both are valid.
One argument, though, would say to keep the service cleaner and don't interact with the UI from the service. (Though there isn't a problem with it.)

Here, we're going to do it in the component. We'll inject the router (private router: Router that needs to be imported from '@angular/router').
Then in the subscribe, in the success case, we want to use navigate:
this.router.navigate(['/recipes']);
This is a great example of programmatic navigation, where we can't use a link in the template, because we want to navigate when some action is done (once the login is complete), not when a user clicks a button.

Next we want to correctly reflect the current status in the UI. The authentication link shouldn't show up if we're logged in, and the recipes link shouldn't display if we're not logged in.

In the header component, we'll want to disable the recies link and the dropdown button if we're not logged in and disable the authenticaion link if we are logged in, and to show a logout button if we're logged in.

We'll need to get the current authentication status into the header component. Do we have a user with a valid token or not?

Right now in the auth service we manage the user with a Subject. This will inform all places in the app about when our user changes. Let's assume it always changes when the authentication status changes. So even if the token expires, the user subject will emit a new value which is then null.
So our header can subscribe to that subject to update correctly. 

So in the header.component.ts file we can inject the authService (and import AuthService from its file path). We then add implements OnInit (also imported from @angular/core), then in ngOnInit we set up a subscription to this.authService.user
ngOnInit() {
    this.authService.user.subscribe();
}

Before the constructor we'll add a new property private userSub: which is of type Subscription (and Subscription is imported from rxjs), and we store that subscription in that userSub property.
this.userSub = this.authService.user.subscribe();

Then we need to add OnDestroy (to the implements and import it).
And in ngOnDestroy (at the end of the class):
ngOnDestroy() {
    this.userSub.unsubscribe();
}

Inside the subscribe we get a user object. This user will be either null, if we're not logged in, or it exists if we are logged in.
ngOnInit() {
    this.userSub = this.authService.user.subscribe(user => {
        this.isAuthenticated = !user ? false : true;
    });
}

We can add a new property above the userSub property, called isAuthenticated with a default of false.
Then in the subscribe where we get a user, we can set this.isAuthenticated to a check. If there isn't a user, we want to set the value to false. Otherwise it's true.
We can shorten that and just write not not user:
this.isAuthenticated = !!user;

Then in the header component we can use that new property to update the UI/template appropriately. 
With ngIf we can make sure that link is only visible if isAuthenticated is true.
<li routerLinkActive="active" *ngIf="isAuthenticated">
    <a routerLink="/recipes">Recipes</a>
</li>

The opposite should be true on the Authenticate Link: *ngIf="!isAuthenticated"

Logout should only be visible if we are authenticated, same with the dropdown.

# adding the token to outgoing request
We have to attach the token to our outgoing requests (to let Firebase know).
In the data-storage service, where we have storeRecipes and fetchRecipes, these requests need to have the token attached to them (the token we're storing in the user object in the auth service).

storeRecipes() {
    const recipes = this.recipeService.getRecipes();
    this.http
        .put(
            'URL',
            recipes
        )
        .subscribe(response => {
            console.log(response);
        });
}

fetchRecipes() {
    return this.http
        .get<Recipe[]>(
            'URL'
        )
        .pipe(
            map(recipes.map(recipe => {
                return {
                    ...recipe,
                    ingredients: recipe.ingredients ? recip.ingredients : []
                };
            });
            )
        )
}

We'll inject the authService into the data-storage service (and make sure we add the import). Then we need to get access to the user in both the store and fetch methods.
We could set up a subscription, but we don't care about every change to the user. We just want to get the token of the currently authenticated user. 
So in the auth service we should also store the token not through a subject, but in a way that we can get on-demand. We can store the token in a variable which is null to start, where we set the token whenever we emit a new user.
However, we can use a different kind of subject. 
the user = new Subject is a subject to which we subscribe and we get info whenever new data is emitted.
rxjs offers a different kind of subject called a BehviorSubject, (also imported from rxjs) and generally behaves just like the other subject: we can call next() to emit a value, and we can subscribe to it to be informed about new values.
user = new BehaviorSubject<User>();
The difference is that BehaviorSubject gives subscribers immediate access to the previously emitted value even if they haven't subscribed at the point of time that value was emitted. That means we can get access to the currently active user, even if we only subscribe after that user has been emitted.
So when the fetch request happens, we get access to the latest user even though when they logged in (before) is when the new user was emitted.
BehaviorSubject needs to be initialized with a starting value, which in this case will be null.
user = new BehaviorSubject<User>(null);
It has to be a user object, and null is a valid replacement, because we don't want to start off with a user.

We don't need to change anything in the auth service; we still emit our user with the next() function, but in our data-storage service now we can reach out to the auth service user and get the currently active user.

So in the fetch method, when we reach out to the auth service, and to the user. We don't want to set up an ongoing subscription, and to make sure we only subscribe once we use .pipe() and a special operator rxjs gives us, the take operator (also imported from rxjs/operators). take is called as a function and we pass a number to it. If we pass one, we're saying we only want to take one value from that observable and then it should unsubscribe.
We're not getting future users, because we're just getting them on-demand whenever fetchRecipes is called.

fetchRecipes() {
    this.authService.user.pipe(take(1)).subscribe()
}

Then in subscribe we get our user object, but only once.
We'll also need to pipe the two observables (the user observable and the http observable) together into one big observable.
We need to add another operator to the pipe for the user observable, exhaustMap. 
exhaustMap waits for the first observable (the user observable) to complete, which will happen after we took the latest user. Then it gives us the user, so in exhaustMap we pass a function; there we get the data from the previous observable, and now we return a new observable which will replace our previous entire observable in that entire observable chain. 
We start with a user observable, and once we're done with that, this will be replaced in that observable chain with the inner observable we return inside of the function we pass to exhaustMap. So in there we'll return the http request. Then we add the other two operators (map and tap) as next steps after the exhaustMap. Then we return the overall observable. This means we can still subscribe to that observable return by retchRecipes. But inside we utilize this user observable to get the user out of it one time.

fetchRecipes() {
    return this.authService.user.pipe(take(1), exhaustMap(user => {
        return this.http
        .get<Recipe[]>()(
            'URL'
        )
    }), 
    map(recipes => {
        return recipes.map(recipe => {
            return {
                ...recipe,
                ingredients: recipe.ingredients ? recipe.ingredients : []
            };
        });
    }),
    tap(recipes => {
        this.recipeService.setRecipes(recipes);
    })
    );
}

For Firebase, we add the token as a query parameter in the url.
We can do it manually, by setting a question mark and then the parameter name has to be auth. 
'URL.json?auth=' + user.token
Or we add a second argument to the get method which is an object where we can set the params. We call the new HttpParams and set the auth equal to user.token. (HttpParams needs to be imported from @angular/common/http)
'URL.json',
{
    params: new HttpParams().set('auth', user.token)
}

If the token is invalid, the request will fail. Later we'll add code to automatically logout a user when a token has expired.

# Attaching the token with an interceptor
We'll create an auth-interceptor.service.ts file in the auth folder.
We need to import Injectable, but we don't provideIn root because we're going to provide it in the module.
We will implement the HTTPInterceptor interface, so we need to import that from @angular/common/http. That forces us to add the intercept method.
export class AuthInterceptorService implements HttpInterceptor {
    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        
    }
}
It needs to get two arguments: the request, the HttpRequest, which is of generic type any; and next, which is our HttpHandler. These types need to be imported. 
Then we'll need to return next.handle(req)

But we want to edit the request and add the token.
So we add the constructor and inject the authService.

Then we reach out the auth service and the user, which is a subject and therefore an observable, and we can subscribe to it. 
We'll need to use the same code as before with the .pipe(take(1), exhaustMap())
We also need to import the operators from rxjs.
Then in that function we return next.handle(req); and we can edit the request based on the user.
Then we'll return the overall chain which will have the handle observable being returned. 

We have to clone the request and update it. 
export class AuthInterceptorService implements HttpInterceptor {
constructor(private authService: AuthService) {}

    intercept(req: HttpRequest<any>, next: HttpHandler) {
        return this.authService.user.pipe(
            take(1),
            exhaustMap(user => {
                const modifiedReq = req.clone({
                    params: new HttpParams().set('auth', user.token)
                });
                return next.handle(modifiedReq);
            })
        );
    }
}

We have to add this in the app module. In the providers array we add an object with the provide keyword and HTTP_INTERCEPTORS which is imported from angular/common/http.
Then useClass: AuthInterceptorService (which needs to be imported). And we set multi to true. 
  providers: [{
    provide: HTTP_INTERCEPTORS,
    useClass: AuthInterceptorService,
    multi: true
  }],

Then in the data storage service (in the fetchRecipes function), we don't add the params, but we do use .pipe() to add the map and tap.

The way it's written now it'll also try to use the interceptor for the signup and login requests - it'll intercept all requests.
It will fail because before a user is logged in, the user subject is null and we aren't able to access the token on user.token if user is null.
We can add a check. If we don't have a user, we just return next.handle(req) and we don't try to modify it.
if (!user) {
    return next.handle(req);
}

We only add the token if we have a user.
Alternatively we can check the url of the request and only add the token for certain urls. 