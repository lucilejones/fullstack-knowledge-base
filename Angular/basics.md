# How an Angular App gets Loaded and Started
The file served by the server is the index.html file
<app-root>Loading...</app-root>
This is one of our own components, auto created by the CLI, the root component of our app; it will tie together our whole app. 
@Component - the decorator

selector: - the selector property assigns a string as a value - 'app-root' 
This is the info Angular needs to replace <app-root></app-root> in the index.html file with the template of the component having this selector. The template of the app component is the content of the app.component.html file.

Angular is a JS framework that changes the DOM at runtime.

# How Angular is triggered
The index.html file has scripts imported at the end of the body. These are injected by the CLI automatically. 
The ng serve process rebuilds our project and creates bundles, JS script bundles, and automatically adds the right imports in the index.html file. 
These script imports will contain our own code too. 
The first code that gets executed is the code in the main.ts file.
In the main.ts file we see that bootstrap gets started by passing an AppModule to the platformBrowserDynamic().bootstrapModule() method.
AppModule refers to the app.module.ts file in the app folder.
In the app.module.ts file we see the @NgModule({}) and there's a bootstrap array to let Angular know the components it needs to be aware of at the time it analyzes the index.html file; and here we reference the AppComponent. 
Angular analyzes the AppComponent, reads the set up passed there, knows about the selector <app-root>, and then is able to handle <app-root> in the index.html file. It will insert the app component and the HTML code (a template) attached to it.

# Components
We build the whole application composing it of components
We'll add/nest other components to the app component.
Each component has its own template (HTML code), maybe its own styling, and its own business logic. 
It allows a complex app to be split up into reusable parts. Each piece is finely controlled. Easy to update and exchange. 

All components we create will be added to the app component html file, not the index.html file.

# Creating a new component
Create a new folder in the app foler
For example, name it server.
Typically each component should have its own folder.

First create the server.component.ts file
A component, simply, is a class, a TS class. Angular is able to instatiate it so we can create objects off of the blueprint. 
We need to export the class. 
export class ServerComponent {}
To start, it is a normal TS class. 
We can't use it just like that, though; we have to let Angular know its a special type of class - a component.
We use a decorator. A decorator is a special TS feature we use to enhance our classes, enhance elements we use in our code.
Decorators are always used by adding at @ in front of them. This one will use the @Component decorator.
This decorator is not something TS knows, so we have to import it: import { Component } from 'angular/core';
Then we pass a JS object to the component decorator to configure it. We can set up important info that will be stored as metadata for this class that will tell Angular what to do with this class. 
One important info piece is the selector - the 'html tag' by which we're able to use this component in another part of the app.  
The selector needs to be a unique string, usually prefixed with app and then a descriptive name. 
example: 'app-server'
The other important piece of info is the template. 
We'll reference another external file, which we first need to create: server.component.html
This will hold the template, the htlm code, of our component. 
We need a relative path for the templateUrl info: 
example: './server.component.html'

In order to use the componet, we'll need to get into the app.module.ts file.

# AppModule and Component Declaration
Angular uses components to build webpages and uses modules to basically bundle different pieces, for exmple, the components of our app, into packages.
A module is a bundle of functionalities of our app. It's also just an empty TS class like our component. We transform it into something else by adding a decorator. It's the NgModule decorator which is also imported from '@angular/core'
There are four properties set up on the object we passed to NgModule: declarations, imports, providers, and bootstrap.
We need to add the new component to the declarations array. 
We also have to tell TS about it. 
So we add an import at the top of the file:
import { ServerComponent } from './server/server.component'
(The file extension is added by webpack when it bundles our app.)

# To use the component
In the app component html file: we'll add <app-server></app-server>

# Creating a component with the CLI
In a new terminal window: 
ng generate component servers
shortcut ng g c servers
This will create a folder with the four files:
server.component.html
server.component.ts
server.component.css - for stying
server.component.spec.ts - for testing
(delete the testing one for now - will go over in a later unit)
It should also auto update the app.module.ts file, but we can double-check that it's added as an import and in the declarations array. 

# Component templates
In the component.ts file we can have either a templateUrl or a template - one points to an external html file, the other if defined inline, in the TS file. But we have to have a template. (Don't always have to have a selector or the styles.)
If we want to write multi-line strings we need to use backticks instead of single quotes.
Using inline might be fine for a little bit of code.
But once we have more than three lines of code, it's better to use an external file.

# Component styles
In the app.component.css file:
We can write noraml css code. 
styleUrls is an array because we can point to multiple style sheets.
We can also use inline style with styles: []
This takes an array of strings with backticks and the css styles we want. 
We can't combine styleUrls and styles. 

# The Component Selector
It works like a CSS selector, we could change it to be an attribute by putting it in square brackets inside a string. selector: '[app-servers]'.
Then we add that attribute to a div instead of putting <app-servers></app-servers>
<div app-servers></div>
Then Angular is selecting the element by attribute and not by the element itself. 
Antother option is to select by class. 
Selecting by id won't work. 
Typically we'll use the element selector style.

# Databinding
Databinding is communication between the TS code of the component, the business logic, and the template, what the user sees. There might be a result in the TS code (like a response from a server or a finished calculation) that we want to display to the user. 
Different ways of communication: output data from the TS code to the HTML template:
string interpolation {{ data }}
property binding [property]="data"
Also sending information (from the user) from the HTML template back to the TS code: event binding (event)="expression"
Combination of both: two-way binding [(ngModel)]="data"

# String Interpolation
A tool for outputting data in a template.
Instead of hardcoding the output.
Example: Server with ID --- is ---.
Server with ID {{ serverId }} is {{ serverStatus }}.
Whatever we have inside the double curly braces has to resolve to a string. We could put in the variable from the TS file, or call a method that returns a string. 
We also can't write multiline expressions. 
A number can be easily converted into a string.

# Property Binding
Often can use either string interpolation or property binding.
We can add a property to the ServersComponent class: allowNewServer and set it to false.
Then we can add the disabled attribute to the button.
However, rather than hardcode, we can have that attribute connected to a condition.
The constructor in the class is a method executed at the point fo time this component is created by Angular. 
In the constructor we can put a setTimeOut() method to set the allowNewServer to true after 2 seconds. 
Then we need to bind it to allowNewServer by putting the disabled attribute (on the button) in square brackets. This lets Angular know we're using property binding, and that we want to dynamically bind some property.
The disbaled property is native (just one of many attributes an element can have), but putting it in square brackets lets us dynamically bind to it. 
After putting it in square brackets, we set it equal to an expression, which in this case resolves to a boolean value. 
It's possible to bind to any of the HTML properties. 
Then this will update dynamically. 

# Property binding vs string interpolation
<p>{{ allowNewServer }}</p> vs <p [innerText]="allowNewServer"></p>
When to use each - if we want to output something in our template, use string interpolation; if we want to change some property, use property binding.
Don't mix property binding and string interpolation.
Putting the property in square brackets makes the entire expression evaluated by Angular as TS. We don't need to put curly braces in there. 

# Event binding
(click) instead of onclick
parentheses are the signal that we're using event binding (similar to square brackets for property binding)
We want to listen to the click event on the button and connect it to a method. 
(click)="onCreateServer()"
Between the quotation marks we put the code we want to execute when this event occurs. We can write a bit of simple logic (chaning a boolean from true to false, for example), but usually we'll want to call a method.


# To know which properties or events of HTML elements we can bind to
console.log() the elements and see which properties and events it offers
Google: YOUR_ELEMENT properties or YOUR_ELEMENT events

# Passing and Using data with event binding
We pass $event to the method we're executing on the input event. 
$event is a reserved variable name we can use in the template
$event will be the data emitted with that event
Then we can use the event.target.value.
(<HTMLInputElement>event.target).value
the HTMLInputElement is needed to inform TS that we know that the type of the HTML element of this event will be an HTML input element.

# Two-way-databinding
Reacting to events in both directions.
For it to work, we need to enable the ngModel directive. We add the FormsModule to the imports[] array in the AppModule.
Then we need to add the import from @angular/forms in the app.module.ts file:
import { FormsModule } from '@angular/forms';
Combining property and event binding - so we also combine the syntaxes: parentheses within the square brackets.
[(ngModel)]="serverName"
We have to use a special directive, ngModel. 
This setup will: trigger on the input event and update the value of serverName; it will also update the value of the input element.

# Directives
