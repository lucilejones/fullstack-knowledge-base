Angular is a JS framework that allows us to create reactive single-page-applications.

SPA: all the changes are rendered in the browser; it's actually one HTML page. JS is much fater than having to reach out to a server for every page change and data update. JS changes the DOM during runtime. 

# VERSIONS
AngularJS (Angular 1) - there were some flaws
Angular 2 - completely rewritten; a totally different framework
There have been several updated versions. They're all the same framework (starting with Angular 2). Angular is pretty backward compatible.

# node
will run in the background to optimize the app
also use npm to install dependencies, etc

# docs for Angular CLI
https://github.com/angular/angular-cli

# to install Angular CLI
npm install -g @angular/cli
(optional to add @latest)

# to create a new Angular project
ng new project-folder-name
(can add --no-strict on the end to disable the TypeScript)
ng new project-folder-name --no-strict

Then cd into the project folder

# to open project in the browser
ng serve --open


# the app component
the html file is the template
the css file is for styling
the ts file is for the definition of the component and what will be converted to JS when it's compiled

in the ts file there's the selector 'app-root' - will be dynamically replaced and updated with our own component

# Modules
Angular is split up into multiple modules (sub-packages) and we need to add them if we want to use a certain feature from them. We'll do that in the app.module.ts file.
We tell Angular which pieces belong to our app.
At the top of the file we add import {} from '@angular/forms';
<!-- the import at the top is a feature of TS because it needs to know where things are and where they come from -->
Then we add it to the imports array to let Angular know we want to import some features.
<!-- Adding it to the imports arry is a feature of Angular because it's part of the @NgModule({}) -->


# Course Structure
Basics
Components and Databinding
Directives
Services and Dependency Injection
Routing
Observables
Forms
Pipes
Http
Authentication
Optimizations and NgModules
Deployment
Animations and Testing

# Installing Bootstrap (locally to a project)
npm install --save bootstrap@3
(version 3 is what Max uses in the Academind course)

# Make Angular aware of the styling package
In the angular.json file:
(In the architect - build node - styles array)
Change: "styles": [
              "src/styles.css"
            ],
To: "styles": [
            "node_modules/bootstrap/dist/css/bootstrap.min.css",
              "src/styles.css"
            ],