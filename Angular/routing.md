# Angular Routing
ability to have different end points for different "pages"
In this example we have a home, servers, and users, and we'd like to only display one at a time.
So we can dynamically load them by clicking on the different links.
/home, /user, etc
We need to configure our app and inform Angular about our routes in the app.module.ts file.
About the @NgModule decorator, we'll add a new const named appRoutes because it should hold all the routes of our application. It should be of a specific type, the type Routes. This type needs to be imported from "@angular/router". The const should hold an array because we will have multiple routes. Right now each rout is just a JS object in the array. The object needs to follow a specific structure for Angular to be able to use it. It always needs a path, what gets entered in the url after our domain. It should be a string. 
const appRoutes: Routes = [
  { path: 'users' } 
];
For example, this first route will go to: localhost:4200/users
We also need to define what should happen when this path is reached. The action typically is a component. We let Angular know that once this path is reached, a certain component should be loaded. And this will be the "page" that gets loaded. 
For this first route we'll use the users component.
const appRoutes: Routes = [
  { path: 'users', component: UsersComponent } 
];
These components that we want to load as "pages" we need to make sure we configure and set up the layout to look like pages.
We can also create an empty path for the home component
const appRoutes: Routes = [
  { path: '', component: HomeComponent } 
];
Then we need to register these routes in our app.
We add the RouterModule to the imports array in teh app.module.ts file.
And make sure to import it from '@angular/router'
With that, we're adding the routing functionality to the app, but our routes are still not yet registered. We need to call the method forRoot() which allows us to register routes for our application. the forRoot() method will receive our appRoutes constant. Now Angular knows out routes.

RouterModule.forRoot(appRoutes)

Next we need some place to render the currently selected component.
The right place to inform Angular where to load it is in the app component template. We get rid of the selectors we already had there except for one - at the place where we want to display the component selected by the route, and instead add a special directive. We don't add the component with its selector.
<router-outlet></fouter-outlet>
It looks like a component but is a directive. (Directives can have any selector. This one has a component-style selector.)
This marks the place where we want Angular to load the currently selected component of our router. 

# Navigating with Router Links
In the app component html file, if we just change the hrefs of the links to the endoint we want (/, /users, /servers) then it will send a request and return a new page, so it's reloading the all each time we click one of those links. We want to not be reloading the page. 
There is a special directive Angular gives us: routerLink
We can use this in place of the hrefs. 
<a routerLink="/">Home</a>
This tells Angular that it will serve as a link, but it will handle the click differently. 
We can also use property binding with routerLink and then pass the path as an array in which we specify all the segments of our path. The first segment is the string, and if there were another / and endpoint, we include this as another segment. This way of doing it allows us to contruct more complicated paths easily.
<a routerLink="/servers">Servers</a>
<a [routerLink]="['/users']">Users</a>

<a [routerLink]="['/users', 'otherendpoint']">users other</a>

This will navigate among the views without reloading the page.
routerLink catches the click on the event, prevents the default (which would be to send a request), and instead analyzes what we passed here to the routerLink directive, parses it and check if it finds a fitting route in our configuration. 
This gives us a better user experience, doesn't restart our app (keeping the app state), and it's faster than reloading the page.

# understanding navigation paths
The slash in front makes it an absolute path
Without the slash it is a relative path - it appends the path we specifiy to the end of our current path (which depends on which component we're currently on).
Relative paths can be useful if we have nested routes. We can also use ./ for a relative path; it's the same as having nothing in front of the endpoint string.
We can even navigate like in directores: '../servers'

# Styling active router links
Giving a visual indication of what the currently active route is.
In our CSS we can have an "active" class.
We'll want to set it dynamically rather than hard-code it. 
We'll use the routerLinkActive directive from Angular.
routerLinkActive=""
We can add it to a wrapping element or to the link itself.
It will attach the class we specify (in the quotes) to that element it sits on. 
In this example we need to put it on the list item since we're using bootstrap.
If we add it to each link they should each receive that class once they're clicked on and active.

However, this way Home is always marked as active. 
Angular marks the path active if it also contains the path that we are on. 
The / is always part of the paths. But we don't want Home to be marked active all the time.
We can add a special configuration to our routerLinkActive directive. 
[routerLinkActiveOptions]=""
It needs property binding because we don't just pass a string. We pass a JS object. 
We can pass exact and set it to true.
[routerLinkActiveOptions]="{exact: true}"
exact is a reserved property on this object, and this will tell Angular to only add the active class if the full path is this link's path. And not if it's just part of the path. 

# Navigating programmatically
If we want to load a route programmatically - if some operation has finished, or the user has clicked a button, for example, and we want to do more than just navigation with the Click. 
If we want to we can trigger the navigation from our TS code.
A button can have a method onLoadServers(), then in the method we do our code and then want to navigate away to another view.
In the constructor of that component we can inject the router. We bind it to a private property (it will be of type Router) and we need to import Router from '@angular/router'.
constructor(private router: Router) {}
With that injected, we can use this.router and then use the method navigate()
this.router.navigate();
This takes an argument that allows us to navigate to a new route, a route is defined as an array of the single or the different elements of this new path. 
this.router.navigate(['/servers']);

# using relative paths in programmatic navigation
Unlike the routerLink, the navigate() method doesn't know which route we're currently on. 
To indicate to the method where we currently are, we have to pass in a second argument which is a JS object. We can add the relativeTo: property. Here we define relative to which route this link should be loaded. By default, this is always the root domain. 
We don't pass a string, instead we inject the route in the constructor.
This will be of type ActivatedRoute (which we need to import from '@angular/router').
ActivatedRoute will inject the currently active route. So for the component loaded, this will be the route that loaded this component and the route is a complex JS object which keeps meta info about the currently active route.
So then we set the relativeTo as a value
{relativeTo: this.route}
With that piece of information, Angular now knows our currently active route, then the first argument can be a relative path. 

# passing parameters to routes
Maybe we want to have a route that loads information for a single user (dynamically) and it'd be helpful to pass the ID of the user to the route.
We can add parameters to our routes, dynamic segments in our paths. 
We do it by adding a colon and then the name we want.
{ path: 'users/:id', component: UserComponent }
We will later be able to retreive the parameter inside of the loaded component by the name we specify here (in this example, id).
The colon tells Angular this is a dynamic part of the path.

# Fetching route parameters
Now we want to get access to the data which is encoded in the URL.
We know we will load the user component and there will be some data in URL for us. 
In the user component TS file we need to inject the active route. (And import it from '@angular/router')
constructor(private route: ActivatedRoute) { }
By injecting this we get access to the currently loaded route.
The currently loaded route is a JS object with a lot of meta data about the currenty loaded route. One important piece of info is the currently active user. 
The ActivatedRoute object we inject will give us access to the id passed in the URL => Selected User
In the User Component we define a user object with an id and name. It is undefined, but we set the structure. 
We can load our user by retreiving this parameter from the URL.
So in ngOnInit, when our component gets initialized we get our user. 
We assign this.user to a JS object with an id and a name. The id can be fetched from our route, and there we have a snapshot property, and on that a params JS object and that's where we get our id. 
We will only have access to properties that we defined in our route parameters.
We defined :id in the route path, so we can retreive the id from the params object.
The name of the user is not encoded in the route right now. So in the route we can add another dynamic part :name
{ path: 'users/:id/:name', component: UserComponent }
Then we can retreive name the same way we retreive id (in the params object).
ngOnInit() {
    this.user = {
        id: this.route.snapshot.params['id'],
        name: this.route.snapshot.params['name']
    }
}

Then in our HTML document we can output that data with string interpolation.
<p>User with ID {{ user.id }} loaded.</p> 

# Fetching route parameters reactively
In this case we can create a button that goes to a new route, but the text on the page will not change becuase Angular will not reinstantiate the component since we're still on the same component.
Angular doesn't know that the data we want changed. 

It's fine to use the snapshot for the first initialization, but we need a different approach to be able to react to subsequent changes. 
In the ngOnInit, after we assign the initial setup, we can use our route object, and instead of using snapshot, we can use the params property that is on the object itself. 
params is an observable - observables are a feature added by some other third party package, not Angular - but are heavily used by Angular and allow you to easily work with asynchronous tasks, and this is an asychronous task. Because the parameters of the currently loaded route might change at some point in the future but we don't know when or if or how long it will take. So we can't block our code and wait for this to happen because it might never happen. An observable is an easy way to subscribe to some event which might happen in the future, to then execute some code when it happens, without having to wait for it. 
(We'll learn more about observables later.)
We can call the subscribe method on params (in the ngOnInit):
this.route.params
  .subscribe()
.subscribe() can take three functions here as arguments
The first one is the most important, it will be fired whenever the parameters change.
It will take an argument of params, of type Params (which needs to be imported).
Params will always be an object which holds the parameters we defined in the route as properties.
In the function body we can update our user object.
this.route.params
  .subscribe(
    (params: Params) => {
        this.user.id = params['id'];
        this.user.name = params['name'];
    }
  );
So now this will update our user whenever the parameters change.
The subscription will be set up with ngOnInit runs, that code will run. But the code to update the user will only run when the parameters change (because it's in the callback function passed to the subscribe method.)
This is the approach to take in order to be safe against changes not being reflected in the template. However, if we know that the component we're on may never be reloaded from within the component (if we know 100% of the time it'll be recreated when it is reached), then we can just use the snapshot and we don't need to subscribe.

# Note about route observables
Angular does something for us in the background. It cleans up the subscription we set up in this component whenever the component is destroyed. 
We could implement OnDestroy, call that after the ngOnInit, save the subscription in a variable called, for example, paramsSubscription of type Subscription (which would need to be imported from 'rxjs/Subscription'). Subscription is not shipping with Angular, but Angular is using that package.
Then we bind the paramsSubscription property to the subscrtiption we set up.
Then in the ngOnDestroy() we unsubscribe:
this.paramsSubscription.unsubscribe();
We don't have to do this because Angular will do this for us for the route observables, but if we add our own observables we will need to unsubscribe. 

# Passing query parameters and fragments
Query parameters are those separated by a question mark, like ?mode=editing
We might also have a hash fragment (#) to jump to a specific place in our app.

In our app module, we add another route that allows us to edit a certain server.
{ path: 'servers/:id/edit', component: EditServerComponent }
On the servers component we need to hook up the links in the list.
On the anchor tag we add:
[routerLink]="['/servers', 5, 'edit']"
(We can make this more dynamic later)
If we want to have a query parameter deciding if we're able to edit the server or not, we add a new property of the routerLink directive. We can bind to the query params property.
[queryParams]=""
Query params is not a new directive; it's another bindable property of the link directive.
We pass a JS object.
We can definte key/value pairs of the of the parameters we want to edit.
[queryParams]="{allowEdit: '1'}"
We can have more than one key/value pair which would then be separated with the & symbol in our endpoint url
We also have the fragment property. We can only have one fragment. 
Here we pass a string in quotation marks (inside the quotation marks) or omit the square brackets around fragment.
[fragment]="'loading'" OR
fragment="loading"
That will add #loading to the endpoint

We can do this programmatically.
In the home component where we have the Load Servers button, if we want to load server 1, then we pass 1 to the onLoadServer() function.
In the TS file we need to adjust the function to take a parameter:
onLoadServer(id: number) {}
It will take the if passed as a number.
Then we'll update the navigate to include the id, and then as a last argument, 'edit'
this.router.navigate(['/servers', id, 'edit']);
And we still want to add query params in the navigate method, so we add an object
this.router.navigate(['/servers', id, 'edit', {queryParams: {allowEdit: '1}, fragment: 'loading}]);

# retreiving query parameters and fragments
In the edit-server component, to get access to our query parameter and fragment.
We'll inject the ActivatedRoute in the constructor and need to do the import (from '@angular/router').
private route: ActivatedRoute
with this added, in the ngOnInit we can retreive our query parameters and the fragment.
There are two ways of retreving it. 
The first way is to access the snapshot.
console.log(this.route.snapshot.queryParams);
console.log(this.route.snapshot.fragment);

Similar to our other situation, this is only run at the time the component is created. So if there is a change to the params from the page we're currently on, we won't be able to react to any changes that happend after the component is loaded.

The alternative is to subscribe to queryParams and fragments as an observables:
this.route.queryParams.subscribe();
this.route.fragment.subscribe();

Angular will handle the unsubscribe when the component is destroyed.

# Common Gotchas
-Loading a single server without an id available - get an error. (We'll learn more about this later - nesting another router in there to have child routing, to load the route dynamically next to our menu.)

-If we parse a parameter from our URl it will always be a string, because out whole UTl is just text. 
We have to convert our id from the params to a number, because our server id is of the type number.
const id = +this.route.snapshot.params['id'];
We also need to do it in the observable.
.subscribe(
  (params: Params) => {
    this.server = this.serversService.getServer(+params['id']);
  }
)

# setting up child (nested) routes
in the app.module.ts file, in our routes, we add another property to the servers route: children
children takes another array of routes, so then we can take the other servers routes and add them inside the array as children. And we remove /servers at the beginning of those routes because it is always prepended.
{ path: 'servers', component: ServersComponent, children: [
  { path: ':id', component: ServerComponent },
  { path: ':id/edit', component: EditServerComponent }
] }

If this is all we do, we'll get an error that Angular cannot find an outlet to load our server component. 
The <router-outlet></router-outlet> is only reserved for routes on the top level.
The child routes of servers need a separate outlet. 
In the server component HTML file we can add a router-outlet (and comment out the code where we were originally loading the app-edit-server and the app-server)
This adds a new hook which will be used on child routes of the route being loaded on the servers component.

We can also do this with the user component and its child route.
It ends up being important that we dynamically update the id, etc because the component wasn't reloaded or exchanged, but we were able to switch the loaded user while the user component was already loaded. 

# congiguring the handling of query parameters
(for example, preserving them if we navigate again)
In the server component we can add another property that we use to configure our navigation. We use the queryParamsHandling property. It takes a string as a value, for example, 'merge' or 'preserve'. This will override the default behavior and make sure the new ones are kept. 

# redirecting and wildcard routes
-404 error handling
-redirecting

We can generate a page-not-found component to be rendered whenever the user navigates to a path we don't have. 
{ path: 'something', component: PageNotFoundComponent }
We can also use redirectTo instead of component, to specify a path
{ path: 'something', redirectTo: '/not-found' }
We can't define all possible wrong urls, we can use the **. This is the wildcard route and tells Angular to catch all paths you don't know.
{ path: '**', redirectTo: '/not-found' }

We need to make sure that generic, wildcard route is the last in our list of routes. The routes get parsed from top to bottom. If it were at the top of the list we would always get not-found.

# redirection path matching
By default Angular matches paths by prefix. It will check if the path you entered in the url does start with the path specified in the route. And every path starts wtih '' (nothing - there isn't any white space there, it's just nothing). 
If we try to use
{ path: '', redirectTo: '/somewhere-else' }
this will not always redirect us. 
To fix this behavior, we can change the matching strategy to "full"
{ path: '', redirectTo: '/somewhere-else', pathMatch: 'full' }
Then we will only get redirected if the full path is '' (so only is we have no other content in the path).

# Oursourcing the route configuration
Typically, if we have more than two or three routes, we add them to their own app-routing-module.ts file instead of putting them directly into the app.module.ts file.
(We'll have a whole lesson later about modules.)

This file will receive the @NgModule decorator.
import { NgModule } from '@angular/core';

@NgModule({
    
})

export class AppRoutingModule {

}

We cut out the const appRoutes: Routes = [] (the whole thing) and add it to the app-routing.module.ts file under the imports and about @NgModule({}).
Then we need to import all the components. We also need to import Routes from the '@angular/router'

we don't need to add declarations in the @NgModule in this file, because the compoenents are already declared in the app.module.ts file. 

We do need to remove the RouterModule from the imports array in @NgModule (and from the imports at the top of the file) in the root app.module.ts file and put that in our app-routing.module.ts file.

Then we need to add our app-routing module back to our main module. For this, we need to add the exports array to the @NgModule (in the app-routing.module.ts file.)
Exports tells Angular what should be accessbile from this module to the module that imports this module. What we want to make accessbile here is our RouterModule.
exports: [RouterModule]
Here, we don't call .forRoot(), because we've already configured it and now we simple export the configured module.
The in app.module.ts, we can import our own routing module.
In the imports array (in @NgModule):
AppRoutingModule
(and make sure to add the import at the top of the file with the file path)

# Guards
-Example use case: we only want certain routes to be available if a user if logged in. We'd want to check that before any of the subroutes are accessed.
-manually checking would be problematic and time-consuming.

# Protecting routes with canActivate
We can run some code at a point of time defined by us.
We add a new folder in our root folder named auth-guard.service.ts

We'll use it as a normal service, so we can export a class named AuthGuard and implement the CanActivate interface. It's provided by the Angular router package and it forces us to have a canActivate method. This will receive two arguments, the ActivatedRouteSnapshot, and the state of the router. 
import {CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot} from '@angular/router';

export class AuthGuard implements CanActivate {
  canActivate(route: ActivatedRouteSnapshot,
              state: RouterStateSnapshot)
}

[my VS code says CanActivate is deprecated]

We'll define that Angular should execute this code before a route is loaded, so it will give us this data, and we simply need to be able to handle it.

canActivate returns either an Observable, which will wrap a boolean and resolve to a true or false value; or it will return a promise, also returning a boolean, or it returns just a boolean.
export class AuthGuard {
    canActivate(route: ActivatedRouteSnapshot,
        state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean  {
        
    }
}

canActivate can run either asynchronously or synchronously. 

For this example we'll make a fake service AuthService (just to show this use case).
In a real application, this might reach out to a server and allow login, logout, and check our current authentication state.

In the auth-guard service we'll add @Injectable to reach out to the other service auth.service to get the functionality from that service into this service. (And import Injectable from '@angular/core')
And in the constructor we'll inject the authService of type AuthService and import from the file path.
And then in the canActivate we'll check whether the user is logged in or not.
We'll reach out to the auth servie for the isAuthenticated method. Then we want to handle it whenever the promise is the auth service resolves. There we'll get back a boolean.
Then we want to check. If authenticated is true, we want to return true; otherwise we want to navigate away because we don't want to allow the user access to that route.
In order to do this, we'll need to inject the Angular router. 
Then we can navigate with the .navigate() method back to our root. 

And we have to return this whole thing (inside the canActivate function):
    canActivate(route: ActivatedRouteSnapshot,
        state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean  {
        return this.authService.isAuthenticated()
            .then(
                (authenticated: boolean) => {
                    if (authenticated) {
                        return true;
                    } else {
                        this.router.navigate(['/']);
                        // we can also return false, though we will prevent the original navigation anyway
                    }
                }
            );
    }

In order to use this guard, we go to the app routing module to define which routes should be protected by this guard.
The routes we want to be protected, we add the canActivate property to it. It takes an array of all the guards we want to apply to this route. It will automatically get applied to all the child routes. 
In this example, in the array we'll add AuthGuard (and make sure to add it to the import list at the top).

So this makes the servers route (and all its child routes) accessible only if the auth guard canActivate method returns true, which will only happen if in the auth service, loggedIn is set to true. 

Before this will work we need to add the two new services to the app.module providers array. 
Then when we try clicking on servers in our app, it will always navigate back to home after the 8 seconds.

# protecting child (nested) routes with canActivateChild
We could grab it from the root servers route and instead put it on each child, but that is not the easiest way. 
We'll import CanActivateChild from '@angular/router'
[my VS code said this is also deprecated]

We'll provide a canActivateChild method in the class. Then we'll add the same logic as the other method.
We can then use a different hook in our routes.
Instead of adding canActivate to the main servers route, we'll add canActivateChild, with the array of services. We still add the AuthGuard.

Then we can click on servers and see the list, but if we try to click on a single server we get redirected back. 

# using a fake auth service
Adding the functionality to let a user log in.
In our home component, we'll add two new buttons: Login and Logout.
We'll inject the authService into the constuctor of the home component, and add the two methods.
For this example here we won't see any visual indication that we're logged in, but we will be able to click on a single server.

# controlling navigation with canDeactivate
control whether we're allowed to leave a route.
Maybe a user accidentally clicked the back button or forgot to click the "update server" button, we can ask if they really want to leave. 
Keeping the user from accidentally navigating away.

In the edit-server component we'll add a new property. Under allowEdit, we'll add a changesSaved property which is set to false to start.
Then we'll want to change that whenever we click on the update server button.
In the onUpdateServer() method, we can add
this.changesSaved = true;

After the changes are saved we want to navigate away. So we need to inject the router in the constructor. 
Then we can add this.router.navigate(['../'], {relativeTo: this.route})
going up one level to the last loaded server. 

Then we want a way to check with the user if they want to leave the route without saving the changes. 
We need to exectute this code in this component because we'll need access to the changesSaved method which informs us on whether the update button was clicked. 
However, a guard always needs to be a service. 
In the edit-server file we'll add a new file called can-deactivate-guard.service.ts

In that file we want to export an interface - a contract which can be imported by some other class which forces this class to provide some logic. 
We'll name it CanComponentDeactivate and it will require one thing from the component which implements it. This component should have a canDeactivate method.
Then we define the type of this method, since it's only an interface and won't contain the actual logic, it will only contain information for what it should look like.
This method will take no arguments, but should return an Observable which will resolve to a boolean or a Promise which will resolve to a boolean or a boolean. (Same pattern as the canActivate method.)
export interface CanComponentDeactivate {
    canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

Then we need to define the class and export it. 
We want to implement CanDeactivate and wrap our our interface.
[Code says CanDeactivate is deprecated.]