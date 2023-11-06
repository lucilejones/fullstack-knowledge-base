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
