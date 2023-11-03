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



