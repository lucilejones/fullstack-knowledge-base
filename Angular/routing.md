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
