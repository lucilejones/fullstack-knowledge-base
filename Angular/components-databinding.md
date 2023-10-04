# separating the app in to compoents
We don't want to have everything in the app.component.
We want to have a leaner index.html file and we want to bundle our business logic.

# how to let the app component know that something changed in one of the child components
# passing data between components


We can bind to HTML elements with native properties and events.
We can bind to directives with custom properties and events. 
We can use it on components, binding to our own properties of our components, binding to custom properties and custom events. 

# Binding to Custom Properties
If we want to access a property element in the server-element.component.html then we need to define it in the server-element.component.ts file. We'll give it a type object. We want it to have a type, a name, and content. 
But it's part of this component only. 
In order to access it in the app.component (to be able to add it to the serverElements array).
By default, all properties of components are only accessible inside these components. We have to be explicit about which properties we want to expose to the other parts of the app. We need to add a decorator to the element property:
@Input() element ...
And we also need to import Input from '@angular/core'
Now any parent component will be able to bind to it.  

# Assigning an Alias to Custom Properties
Sometimes we don't want to use the same property name outside of the component as we use inside of it. 
Inside the component we might use element, but then outside of the component we use srvElement.
We can pass an argument to the @Input() with the property name as we want to have it used outside:
@Input('servElement') element: ...

# Binding to Custom Events
Passing data up to a parent component
We set up a custom event, serverCreated which calls the onServerAdded method and receives an event object.
We pass that object, serverData, to our onServerAdded method in the app.component.ts file.
We get the data in onServerAdded and we call onServerAdded once our custom event occurs on the app.cockpit component. This event should give us the data we expect and we catch it with $event.

Then in the app-cockpit component we need to emit our own event.
We're waiting for events called serverCreate and blueprintCreated.
So in the app-cockpit component we create two new properties: serverCreated and blueprintCreated.
Instead of putting @Input in front of them, to make them events, we need to assign a value new EvenEmitter. We also need to import EventEmitter from @angular/core.
EventEmitter is a generic type, <>. In the angle brackets we define the type of data we're going to emit.
In this example it will be an object with a serverName and serverContent.
We also have to add parentheses at the end to call the constructor fo eventEmitter and create a new eventEmitter object which is now stored in serverCreated.
serverCreated = new EventEmitter<{serverName: string, serverContent: string}>();
EventEmitter is an object in the Angular framework that allows us to emit our own events.

Then in our function onAddServer we can call serverCreate with the emit method:
this.serverCreated.emit();
This will emit a new event of this type. 
We pass the object, with the serverName and the serverContent. 
  onAddServer() {
    this.serverCreated.emit({
      serverName: this.newServerName,
      serverContent: this.newServerContent
    });
  }

We added @Input to make a property bindable. Now we need to add something to serverCreated and blueprintCreated to make them listenable.
It's also a decorator: @Output()
We're passing something out of the component.

# Assigning an Alias to Custom Events
Similar to @Input(), we put the alias inside the parentheses.
@Output('bpCreated') blueprintCreated = new ...
Then in the html file, that's the event we listen for:
(bpCreated)

# Some use cases where the distance between two components that should talk to each other, the chain of inputs and outputs gets too complex.
In those cases we'll use services.