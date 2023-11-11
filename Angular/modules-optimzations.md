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
Feature module - grouping together the parts of the app that are used together in a certain area of the app.
Example: ProductsModule with ProductsListComponet and ProductComponent, plus an OrdersModule with OrdersComponent and HighlightDirective

In the reipces app, Max divides it into Recipes, ShoppingList, and Auth.
There's also the header and the shared folder.

In the recipes folder, we create a recipes.module.ts
We export a class and name it RecipesModule {}
The class needs @NgModule as a decorator (imported from @angular/core)
The @NgModule decorator takes an object that we pass into configure that module, and then we can add what we had previously put in the app module.
@ngModule({})
We'll include a declarations array and add all the recipe-related components. 
We remove them from the app module and put them in the recipes module.
declarations: [
    RecipeComponent,
    RecipeListComponent, etc.
]

We can also remove the unused import statements. We don't need to import the types into the app module anymore. But we do need to import them into the recipes module.

Then we need to add the exports array and add all the components to that array, so they will also be available to any module that imports this module.

In the app.module.ts we add RecipesModule to the imports array there. (And also add the import statement at the top: import { RecipesModule } from './file-path';)
The imports array in @NgModule is there for Angular to add features of other modules into this module. The import at the top of the file is a TypeScript feature to tell TS where to find the type.

If we try to compile at that point we'll get an error: 'router-outlet' is not a known element. We're using 'router-outlet' in the RecipesComponent.
The RouterModule is imported into the app-routing module in order to configure our routes and add routing features, like the RouterLink directive and the router-outlet directive.
We export it in the exports array here in app-routing.module.ts, and it is imported into the app module. However, it won't automatically be available in the recipes module.
Everything in a module stands alone.
The recipes module and the components there have no access to all the things we import into the app module. (The routing module, the forms module, etc.)

# Splitting modules correctly
One way to fix that error is to import the RouterModule (without .forRoot()) in the imports array of the recipes module.

Before now everything was getting access to every feature from the app module. 
ngIf and ngFor are provided by the browser modules, form directives by the forms module, etc.
[the http client module works differently becuase it only provides services and those are available application-wide]

One special case: the BrowserModule must only be used once in the app module. It does general application startup that only runs once.
In order to get access to ngIf and ngFor, etc in other modules, we import the CommonModule. (and import that from '@angular/common')

We also need to import the ReactiveFormsModule into the RecipesModule.

Then everything should be running as before.

# Adding routes to feature modules
We can move the recipes-related route configuration away from the app-routing module and into the recipes module.
In the imports of the recipes.module.ts, we can add .forChild() to the RouterModule.
.forRoot() is only used once in the app-routing module.
In a feature module which we plan on importing into the app module, we use .forChild() which will automatically merge the child routing configuration with the root routes. Then we can pass in the same array of route configurations we passed to .forRoot().
To keep the file leaner, we can create a separate recipes-routing.module.ts file.
We can declare the routes, const routes: Routes = [], then we cut the recipes route from the app-routing module and paste it in here. Then everything would need to be imported at the top of the recipes-routing file (components, authguard, etc).

Then in the @NgModule, in the imports we list RouteModule (which also needs to be imported from @angular/router) we can use .forChild(routes) and then pass in the routes.
We also need to export the RouterModule. 
Then in the recipes module we import the RecipesRoutingModule

And since we import the RecipesModule into the app module, we should stil have the routes in the recipes routing file as part of the general routes configuration.

# Component Declarations
In the routes, we have the RecipesComponent loading whenever we visit /recipes.
The RecipesComponent is also declared in the RecipesModule.
We have to add all the components to the declarations - whether we're going to load them via the routing or a template.

If we define which components should be loaded for which route, we don't need to export them in the recipes module. We're only using them internally in the recipes module. We have them either embedded into other components or are loading them through the recipes routing.

# The ShoppingList feature module
We don't have to have a separate shopping-list routing file. We can just put that in the shopping-list.module.ts file:

imports: [
    RouterModuel.forChild([
        { path: 'shopping-list', component: ShoppingListComponent },
    ])
]

We also need to import the CommonModule and the FormsModule.

Then in the app.module we need to import the ShoppingListModule

# Understanding Shared Modules
If we have two feature modules that use a lot of the same things, we can avoid code duplication and have leander modules.
We can put shared features (components, directives, etc) into a shared module and then import that module into both the other feature modules.

We can have multiple shared modules, so we can name that by which features we're grouping together, but if we just have the one it can be shared.module.ts
We'll declare the AlertComponent, the LoadingSpinnerComponent, the PlaceholderDirective, and the DropdownDirective.
We'll import the CommonModule and export all the components, directives, and the common module.
Then wherever we import the shared module, we have access to all these features that we initialize here.

Then in the shoppinglist module we can replace common module with the shared module.
It's not super helpful in this example because we just replace one module with another; but if we wanted to use the alert or the dropdown in the shopping list module, we'd gain a lot more from that.

We can do the same thing in the recipes module.

Keeping this just as it is, we get an error saying that the DropdownDirective is part of the declarations of two modules: SharedModule and AppModule.
This is an important concept: we can only define or declare components, directives, and pipes once. We can import a module into multiple imports. 

So in the app module we need to remove from the declarations all the components and directives we declared in the shared module.
Also we need to cut entryComponents from app module and add it shared module.
Then we import the shared module to the app module.

# understanding the core module
All modules are created the same way with NgModule, but there are different types based on what we put in them and how we use them.
The core module can make the app module a bit leaner.
For example, we can create a core module to hold/handle two services (to take them out of the app module), and then we just import the core module into the app module.
Or we can use @Injectable with providedIn: 'root' and then we don't need to provide them in the app module or core module.
(Max recommends providedIn)

In the recipes app, in teh app.module, there are the ShoppingListService, the RecipeService, and the HTTP Interceptor. (There's also a DataStorageService that's just providedIn: 'root' in the data-storage-service.ts file - that one doesn't need to be in the app module or core module; it is still available application-wide)
The core module provides these in a separate module to have a place to see all the core services of the app.

Interceptors, however, have to be added to the providers array.
We don't need to export services, they're automatically injected on the root level.
Then in the app.module, we import CodeModule.

# adding an Auth Feature Module
Now in the app.module we do still need to keep the HttpClientModule because it sets up some global services (the injectbale HTTP service), and we also keep the app-routing module and the other modules we created.

# understanding Lazy Loading
So far what we've done separating in the different modules, it just helps with making the files leaner and maintaining code, etc. But it won't actually influence performance.
Using multiple feature modules is a prerequisite for lazy loading.

Right now, when we visit any page, we're loading everything - all the page views.
It'd make more sense to only load pages when we actually visit them.

With lazy loading, we only initially load the root route content - only the app module code and the code of the components registered there. And we don't load the other modules. Only when we visit another route/module.
We initially download a smaller code bundle, and we dowload more code when we need it.

# implementing Lazy Loading
If we're using Angular 8+, there's a new way to specify lazy-loading routes.
Instead of:
const routes: Routes = [{
  path: 'your-path',
  loadChildren: './your-module-path/module-name.module#ModuleName'
}];

We use:
const routes: Routes = [{
  path: 'your-path',
  loadChildren: () => import('./your-module-path/module-name.module').then(m => m.ModuleName)
}];

Then in the tsconfig.json file we need to use
"module": "esnext", instead of "module": "es2015",

This syntax replaces the 'string-only' approach and will give better IDE support.

For lazy loading to work our feature module needs to bring its own routes, its own route congif, with .forChild().
In the recipes routing file, we change the recipes path to an empty string: ''
Then we'll add that path back into the app routing module: 'recipes'
The recipes routing module needs that empty string path because we're already on the /recipes path that we laid out in the app.module. We don't want to end up with /reipces/recipes

Then instead of adding a component, we add loadChildren which is a special property in the route config; it tells Angular to only load the content and module this points at when the user visits this path.
Then we set loadChildren to a string which is the path we want the module to load.
Then at the end we need to add the name of the Module.
Angular will go to that path and then try to dynamically import a specific object from that file; we have to tell Angular the name of the export class in that file (ex. RecipesModule). We put a hash before the module name.

Now the code will be split at that point. So everything at that path - the module and everything the module uses - will be put into a separate code bundle which is then downloaded and parsed on demand as soon as a user visits that page but not sooner.

The newer syntax for loadChildren doesn't take a string, but an anonymous arrow function. In the function body we call import (dynamically as a function). The to import we pass the path to the module. And import resolves a promise, so we can call .then(). There we pass in a function which receives the module (so we use m for the argumant), then on the module object we can extract the RecipesModule.

After any changes to the lazy loading we'll want to restart the ng serve for it to have an effect.

We get an error because we're still importing the RecipesModule into the app module. If we have it set up with lazy loading in the routes module, we need to not include it in the imports array in the app.module.ts. (Otherwise we're trying to load the RecipesModule both eagerly and lazily).

# more lazy loading
Doing the same thing for the shopping-list module and the auth module.

# preloading lazy-loaded code
The way we have the code set up right now, the code is downloaded and parsed right when we need it, which leads to a very tiny delay.
We can tell Angular to preload lazy-loaded modules to avoid this delay.
We go to our root router module (in this project in the app-routing.module.ts) and pass a second argument to the RouterModule.forRoot(); it's an object where we can configure the root router and we can set up a preloading strategy. The deafult is no preloading, so we can change it to PreloadAllModules. (Which also needs to be imported from the Angular router.)
RouterModule.forRoot(appRoutes, {preloadingStrategy: PreloadAllModules})

Now Angular will still split the code into separate bundles, but it'll preload the other modules as soon as possible.
Then the initial bundle is still kept small so the initial download is fast, but then when the user is browsing that first page, we preload these additional code bundles so later navigation requests are faster.
(We can also add our own preloadingStrategies the maybe specify only preloading certain modules, etc)

# Modules and services
Where we provide services affects the instances of the services we're working with.
We can provide services in the app module, in the app component or other component, we can add services to providers of any eager-loaded modules, and we can add services to providers of lazy-loaded modules. And we can provide services by adding the providedIn - and that's the recommended method for all services we plan on providing in app.module otherwise.

When we provide a service in the app module or with providedIn root, the service - the same instance of that service - is available application-wide. 
If we provide a service inside a component, the service is only available for dependency injection inside of that component tree; and all these components will share the same instance. Even if we provide it to siblings, each sibling component tree has access to its own service instance. 
If we add a service to providers of an eager-loaded module, everything is bundled together initially, so those services will be available application-wide with the same instance.
If we add a service to providers of a lazy-loaded module, there the service is only available in that loaded module and it gets its own instance. (Even if we provide the service in the app module and in the lazy-loaded module, the lazy-loaded module will get a separate instance.) This might be desired behavior, it might not.

When we provide a service in the app module or in @injectable, we use the root injector (the root dependency injection mechanism of Angular). When we inject it in a component, we use the component-specific injector which is not the root injector.
In an eager-loaded module the root injector is used, but with a lazy-loaded module a separate child injector is created by Angular for that module and that module has its own instance of the service. 
The default should be that we use the app module or @Injectable.
There might be use cases where we only need to provide a service for a certain component tree.
Eagerly-loaded modules should pretty much never provide services.
In a lazy-loaded module we can add a service to providers if we deliberately want to have a separate instance of the service there. Otherwise we can have strange bugs.

# Loading Services

# Ahead-of-Time Compilation
Ahead-of-Time (AoT) vs Just-in-Time (JiT)
One other optimization technique we should perform before we ship our app to production - before we deploy the app to a server.
Angular will parse our templates and then update the real DOM based on the instructions in our templates (the templates have Angular-specific code).
JS and the browser don't understand our templates.
When we use the TS compiler to compile our TS code into JS code (which the browser understands), we also have another compiler in that build process working behind the scenes as part of ng serve, which is the Angular compiler (included in the Angular framework). THe Angular compiler compiles template syntax into JavaScript DOM instructions. It takes the compiled JS code and our templates and translates all the logic data into concrete instructions that update the real DOM. This by default happens all in the browser, becuase the Angular compiler is part of the Angular code we're shipping. 
This process is called Just-in-Time compilation. Because the Angular templates are compiled just in time when the rendering occurs in the browser.
The obvious downside is that this process takes time, and the Angular compiler is actually quite large.
Instead, we can do that compliation during development as part of the build process and then not have the Angular compiler be part of our app.
That process is called Ahead-of-Time compilation.
The Angular template compiler runs during the build process before the app is deployed.

The default JiT compilation is better for debugging and fast updating of the running application.
But for production and uploading to a server, we want to optimize our code as much as possible and shrink it to as small of a bundle as possible.
We do that with the command: ng build --prod
This command takes our Angular application and instead of spinning up a development server, but builds the entire app into a few files which we can then deploy.
This command uses the Ahead-of-Time compilation. 

We do get an error (the JiT compiler is a bit more forgiving than the AoT compiler).
Max moves some code out of the template into the TS code.

The build command will create a new dist folder - it has the project name and files with the content of our app, but now bundled and optimized.


# official docs:
https://angular.io/guide/ngmodules
