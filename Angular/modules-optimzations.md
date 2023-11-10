# What are modules?
Angular modules - NgModule, like the app module or the app routing module.
Ways of bundling Angular building blocks together - components, directives, services, pipes.
We need to group them together so Angular is aware of these features. Angular doesn't automatically scan all the files in our project.
Every Angular app has to have at least one module, the app module.

Angular analyzes NgModule to "understand" our app and its features.
Angular modules define all the building blocks: components, directives, services, etc.
An app requires at least one module, but can be split into multiple modules.
Core Angular features are included in Angular modules (e.g. FormsModule) to lead them only when we need.
We can't user any feature of building block without including it in a module.

# analyzing the AppModule
So far in the recipes app we have the app.module.ts and the app-routing.module.ts. They both have the @ngModule decorator.

- declarations is an array of all the components, directives, and custom pipes we're using. We have to declare them or we can't use them in the app.
- imports - allows us to import other modules into this module
We use a few modules that ship with Angular (FormsModule, HttpClientModule), and our own AppRoutingModule
For example, the FormsModule has a declarations array with all the directives we use with forms. 
- providers - we list all the services we want to provide; any service we plan on injecting needs to be listed here (or, the alternative is to use the @Injectable({providedIn: 'root'}) in the component where we want to use it.)
- bootstrap - important for starting our app; it defines which component is available right in the index.html file. We typically have one root component.
- entryComponent - when we're creating components programmatically

The app-routing module holds the route configuration. We could put all the routes in the app.module, but we want to keep the app module a bit leaner. 
The app-routing module imports the RouterModule from Angular and uses the forRoot method. Then we export the RouterModule from the app-routing module.
Every module works on its own.
We need export the app-routing module so we can then import it into the app module. Otherwise it would only be available in the app-routing module.
When we import a module we import everything that module exports.

The bigger our application grows, the more likely we'll want to split it up into separate modules. This can also help with performance. 

# Getting started with Feature Modules