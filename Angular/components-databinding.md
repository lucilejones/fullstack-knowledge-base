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

# View Encapsulation
CSS is normally applied to the whole application (if in an app.css file, for example), but there is a behavior enforced by Angular (which is not the default behavior of the browser)  - they are encapsulated with the component they belong to. 
We can define the CSS for each separate file. 
Elements will get an attribute applied by Angular to enforce the style encapsulation. It gives the same attribute to all elements in a component. _ngcontent-ejo-0

We can override it: in the element component.ts file, in the @Component({}) decorator we can add a property called encapsulation. There, as a value, we can give it ViewEncapsulation (which needs to be imported from '@angular/core), and then we choose one of three options. Emulated is the default, so we don't need to choose that one. None will make it so Angular doesn't give the elements in that component the unique attribute. Then styles defined there will get applied globally, also affecting other components. 
import { ViewEncapsulation } from '@angular/core';
@Component({
  ...
  encapsulation: ViewEncapsulation.None
})

If we choose Native (or now called 'ShadowDom'), this uses the ShadowDom technology.

# Using Local References in Templates
To use instead of two-way binding. <input [(ngModel)]="newServerName">
Instead, we can place a local reference on an element. <input #serverNameInput>

A local reference can be placed on any HTML element. We add with a # and the name of our choice. This reference will hold a reference to this element, not to the value we enter, but to the whole HTML element. 
We can use the reference anywhere in the template, but not in the TS code. 
We can pass the reference as an argument to the (click)="onAddServer(serverNameInput)" on the Add Server button. 
And that's how we can use it in the TS code - passing it in the onAddServer function and then receive it in the TS code onAddServer(nameInput) in the TS file. 
We can let TS know it's type (HTMLInputElement) and then use the nameInput.value to set the serverName.
Local reference allows us to get access to some elements in our template and then use in the template or pass it on and use in the TS code.

# Getting access to the template and DOM with @ViewChild
For Angular 8+ instead of:
@ViewChild('serverContentInput') serverContentInput: ElementRef;
We'll use:
@ViewChild('serverContentInput', {static: true}) serverContentInput: ElementRef;

If we're accessing the selected element inside of ngOnInit(), the same change (add { static: true } as a second argument) needs to be applied to all usages of @ViewChild() (and also @ContentChild() which we'll learn later).
If we're accessing the selected element anywhere else in the component (not inside ngOnInit), we need to set static: false instead.

For Angular 9 or higher, we can omit static: false and only need to specify static: true if we're using the selected element inside ngOnInit().

Sometimes we want to get access before/without calling a method. 

In the template, we use #serverContentInput
In the TS file we have a property: serverContentInput;
We add @ViewChild() in front of it, and pass it an argument - the selector of the element. We can pass as a string with the name of the local reference. 
Another option would be to pass @ViewChild a component (without quotes) to get access to the first instance of that app component. 
The serverContentInput is of type ElementRef.
We also need to import ElementRef from '@angular/core'
The element has a nativeElement property that allows us to get access to the underlying element. Which has a value because it will be the input element. 

It's not recommended to use this option for accessing the DOM (for maniputlating or changing the elements). Angular has a better way that we'll learn later.

# Projected Content into Components with ng-content
Initially we have *ngIf in the server-element component check whether we have a type server or type blueprint. However, if we want to check that in the app component instead, we need to change it a little bit.
If we want to add it between the opening and closing tags of the <app-server-element></app-server-element> (which is our own component in the app component), by default all that content will disappear from our app - what's seen on the browser. 
We can use a directive to change that default behavior. It looks like a component but doens't have its own template: <ng-content></ng-content>
In the server-element-component template, in the place where we want to render the content, we put <ng-content></ng-content>
It serves as a hook we can place in our component to mark the place for Angular where it should add any content it finds betweent the opening and closing tag of that element. 
It will be projected into the server element component. 
This will be a good solution (better than property binding) when we have more complicated HTML code that we want to pass down to a component. 

# Understanding the component lifecycle
Once a new component is instantiated, Angular goes through a couple of different phases in this creation process, and it will gie us a chance to hook into these phases and execute some code. 
Lifecycle hooks:
- ngOnChanges - called after a bound input property changes
It's exectued right at the start, when a new component is created, but also always called whenever one of our bound input properties changes. Properties decorated with @Input(), whenever they receive new values.

- ngOnInit - called once the component is initialized
Gets executed once the component has been initialized - it hasn't been displayed or added to the DOM yet, but the basic initialization is finished, our properties can be accessed, etc. The object was created. ngOnInit will run after the constructor. 

- ngDoCheck - called during every change detection run
This will run whenever change detection runs - the system by which Angular determines whether something changed inside a component, if it needs to change something in the template. To re-render that part of the template. This will run on every check, not just if something changed, but when there was a click or a timer or other event. Even if nothing changed.

- ngAfterContentInit - call after content (ng-content) has been projected into view
This is called whenever the content which is projected via ng-content has been initialized. Not the view of the component itself, but the view of the parent component. 

- ngAfterContentChecked - called every time the projected content has been checked
Executed whenever change detection checked this content we're projecting into our component. 

- ngAfterViewInit - called after the component's view (and child views) has been initialized
Once the view of our component has been finished initializing and has been rendered.

- ngAfterViewChecked - called every time the view (and child views) has been checked
Called whenever our view has been checked, once we're sure all changes were displayed or no changes were detected.

- ngOnDestroy - called once the component is about to be destroyed
Called right before the object will be destroyed by Angular.

# Lifecycle hooks in action
Callig the hook inside the class is enough, but it's a good idea to use the implements ... in order to be explicit about which interfaces the component uses and which methods our component will have. 
(We do have to import all the interfaces.)

ngOnChanges is the only hook that receives an argument, which is of type SimpleChanges and also needs to be imported from '@angular/core.
ngOnChanges(changes: SimpleChanges){} 
constructor gets called, and ngOnChanges gets called, before ngOnInit
The SimpleChanges is an object which has an element which is of type SimpleChange; element is our bound property from the @Input.
Angular gives us some info: the currentValue, is this the first change? yes, and the previousValue (which in the first case is undefined)

# Lifestyle hooks and template access
If we want to get access to the heading in the server-element component, for example, we can place a local reference named #heading on it, and place a new @ViewChild property in the class, with the selector 'heading'. It has the type ElementRef which also needs to be imported from '@angular/core'.
We won't be able to access it in the ngOnInit console.log, but we will be able to access it in the ngAfterViewInit console.log.
AfterViewInit gives us access to the template elements, but before that hook has been reached we can't access them. We can't check the value of some element in the DOM because it hasn't been rendered yet.

# getting access to ng-content with @ContentChild
in Angular 8+ - 
instead of: @ContentChild('contentParagraph') paragraph: ElementRef;
we need to do: @ContentChild('contentParagraph', {static: true}) paragraph: ElementRef;
If we access the selected element anywhere else in teh component that isn't ngOnInit, we need to set static: false instead.
For Angular 9 or higher we can omit static: false, but do need to specify static: true when using the selected element inside of ngOnInit().

In the app component where we project content into the server element, maybe we also want to place a local reference on the paragraph element. #contentParagraph
If we want to use that in our server element component, which is where this content will end up, we can't use @ViewChild because it's not part of the view, it's part of the content. 
So instead we use @ContentChild (which also needs to be imported from '@angular/core').
We pass it a selector 'contentParagraph'. And then we store it in a property, paragraph, which will be of type ElementRef.
We won't be able to reach the value or anything before we reach ContentInit.
We can use @ContentChild to get access to content which is stored in another component but passed on via ng-content.
