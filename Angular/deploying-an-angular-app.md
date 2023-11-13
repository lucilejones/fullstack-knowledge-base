Deploying to a server, with a domain, and people can visit it on the web.

# deployment preparation and steps
Use and check environment variables
Polish and test code
Build the app for production: ng build --prod
Deploy build artifacts (generated files) to a static host
(A static host is a web server capable of serving HTML, JS, and CSS, but that's not capable of running any server-side language like PHP or NodeJS.)

# Using environment variables
In the src folder we have an environments folder. 
There are two files in there: environment.ts and environment.prod.ts

We have an exported constant in the environment.ts file:
export const environment = {
    production: false
};
(In the prod.ts file the const is set to true.)

We can add key/value pairs to that constant.
For example, an API key.

The Angular CLI will automatically swap these two files when we're building for production. We don't have to write any code for that behavior to occur.

In the recipes project we're sending requests to the Firebase back-end using an API key.

firebaseAPIKey: 'string'

Then in the auth service we import the environment values.
import { Environment } from '../../environments/environment';
And we use it like this:
'string?key=' + environment.firebaseAPIKey

# Example: Deploying to Firebase Hosting
ng build --prod
It compiles the TS code to JS code.
It compiles as the templates to JS instructions that contains all the logic for updating the DOM.

We'll get a dist folder, and in there we have a folder of the project name. In that folder we'll have assets, the main polyfill, runtime files, and other files, including the HTML file, etc.

We can google static website hosts.
AWS S3, and also Firebase.

Firebase:
Install the Firebase CLI.
npm install -g firebase-tools

Then run firebase login
(Will need a firebase account)
run firebase init 
(to connect this project here with one of our Firebase projects)

Then there are options and we choose the Hosting one.
Then we choose the project we created - we need a Firebase project.

Then we need to choose a file for the public directory. We don't want to use the default (public), but we'll use dist/ng-project-name
Then asked if we want to congifure as a sinlge=page app and we choose y for yes.
The next question is to overwrite the existing HTML file and we choose no.

Then run firebase deploy
Once it's finished it'll give us a hosting URL


When deploying it's important to make sure our server is configured to always serve the index.html file.
