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