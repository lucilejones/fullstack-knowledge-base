# What are dynamic components?
They are components we create dynamically at runtime. For example, an alert, a modal, an overlay - something which should only be loaded on a certain action.
Dynamic components are not a specific feature of Angular, but we can create components that we load through our code.
We'll learn how to create this kind of component, how to load it on demand, how to communicate with it, and how to get rid of it.

# Adding an alert modal component
In the recipes project:
We can change our error message to be a dynamic component.
We can make an alert box presented as an overlay, and then we click ok to get rid of it.
We can make a new folder, alert, in the shared folder, and name it alert.component.ts
The template will have both the backdrop and the overlay box (we could make them into separate components, but this way is fine).
We'll write some custom CSS.

The basic HTML:
<div class="backdrop"></div>
<div class="alert-box">
    <p>{{ message }}</p>
    <div class="alert-box-actions">
        <button class="btn btn-primary">Close</button>
    </div>
</div>

Then we need a css file.

In the alert.component.ts file we need to add the message property. It'll be a string, and it should be settable from outside. We will use @Input() (which needs to be imported)
@Input() message: string;

We need to add the component to the declarations arry in the app.module.

In the component template where we want the alert to show up, we can use the selector: <app-alert></app-alert>
And there we have to set the message property, which we made bindable with @Input.
Then we'll show the entire box if we have an error.
<app-alert [message]="error" *ngIf="error"></app-alert>

The CSS:
.backdrop {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.75);
    z-index: 50;
}

.alert-box {
    position: fixed;
    top: 30vh;
    left: 20vw;
    width: 60vw;
    padding: 16px;
    z-index: 100;
    background: white;
    box-shadow: 0 2px 8px rbga(0, 0, 0, 0.26);
}

.alert-box-actions {
    text-align: right;
}

We could go more advanved and use ng content to pass dynamic DOM into the box, etc. But what we have works for this example. 

# Understanding different approaches
-dynamic components are loaded programatically
-one way: *ngIf
Uses a declarative approach; we add that component selector in the template and then use ngIf to only load it upon a certain condition.
The omponent is embedded via selector (declaratively); and *ngIf controls whether the component is added to the DOM.

-another way: in the past was named dynamic component loader; a helper utility that doens't exist anymore. Now there's the general concept of creating a component in code and then manually attaching it to the DOM.
So we control how the component is instantiated, how data is passed into it, and how it is removed.
The component is created and added to the DOM via code (imperatively); the component is managed and added by the developer.
This approach can be useful; also we can control it entirely from the code without having to touch the template.

# Using *ngIf
The way we've set up our alert, it's included with its selector and controlled with ngIf. 
To close the alert, we need to emit and event and make that event listenable from outside. (@Output, which needs to be imported from @angular/core).
This variable, close, will hold a new EventEmitter (which also needs to be imported), we'll add void as the type because we're not emitting any data; 

In the AlertComponent:
@Output() close = new EventEmitter<void>();

Then we need a method (which will be triggered whenever a user clicks the close button or the backdrop):

In the template:
<div class="backdrop" (click)="onClose()"></div>...
<button class="btn btn-primary" (click)="onClose()">Close</button>

Then in the onClose we can use the close event we created and emit it.

In the app component (or whichever component has the <app-alert>) we can use that close property and call a new method, onHandleError()
Then we create that method in the app.component.ts
  handleError() {
    this.error = null;
  }

This sets the error back to null. Then the condition for displaying the error will be removed. 
[For my temporary example, I used the toggleError function I made instead of handleError]
<app-alert [message]="error" *ngIf="error" (close)="toggleError()"></app-alert>

There are rarely situations where we'll need to imperative approach. The ngIf is much simpler and more straightforward.

# Preparting programmatic creation
If we want to user the imperative method, first we need a new method in the app component that shows the error.
private showErrorAlert(message: string) {

}

Then when we set the error we have to call this.showErrorAlert(); then we wouldn't set the error globally on a property with this.error = errorMessage. Instead we pass the errorMessage to the this.showErrorAlert.
Then we need to manually instantiate a component. We can import AlertComponent.
We cannot create our own component like this (using the component as a class; it will make a JS object, but not a componet; it's valid TS code but not valid Angular code):
const alertCmp = new AlertComponent(); 
This will not work.

Instead we need to let Angular create the componet. We use the component factory, which needs to be injected in the constructor. (It will be of type ComponentFactoryResolver, imported from Angular)
private componentFactoryResolver: ComponentFactoryResolver

It has to be the ComponentFactoryResolver, not the ComponentFactory itself.

So then we use this.componentFactoryResover which has access to the method resolveComponentFactory and we have to pass the type of our component. This method will return a component factory, not the component itself.

private showErrorAlert(message: string) {
    const alertCmpFactory = this.componentFactory.resolveComponentFactory(AlertComponent);
}

So then this is an object that knows how to create alert components.
Then we need a place where we can attach it in our DOM.

Angular needs a view container ref - an object managed by Angular which gives it a reference to a place in the DOM with which it can interact; it also has methods like "create a component here".
To get access to it we create a helper directive.
In there we export a class; and the directive needs a selector: '[appPlaceholder]'
This directive will inject the view container ref (imported from angular/core).

export class PlaceHolderDirective {
    constructor(public viewContainerRef: ViewContainerRef) {}
}

Then we can add the directive somewhere in our DOM, and then get access to it with  @View Child, and then get access to the public viewContainerRef. 

# creating a component programmatically
Then in the app component, we add <ng-template></ng-template>
This will not render anything to the DOM, but still is accessible in the Angular templating language.

<ng-template appPlaceholder></ng-template>

Then in the TS file, we use @ViewChild to get access to that directive, in there we pass in a selector. If we pass the placeholder directive as a type, Angular will find the first place where we use that directive in the template.
@ViewChild(PlaceholderDirective)

[Later versions use @ViewChild(PlaceholderDirective, {static: false})]

Then we can store this in a property of that component, alertHost and set it to type PlaceholderDirective
@ViewChild(PlaceholderDirective) alertHost: PlaceholderDirective;

Then alertHost can be used in the showErrorAlert method.
const hostViewContainerRef = this.alert.viewContainerRef;

We need to clear anything that might've been rendered there before:
hostViewContainerRef.clear();

Then we can use our component factory to create a new alert component. This method doesn't need the type, but needs to be passed the factory. And that will create a new component.

hostViewContainerRef.createComponent(alertCmpFactory)

With just this much we'll get an error. We need to add the factory to @NgModule.entryComponents [but Angular 9 or higher will not give this error.]

# Understanding entryComponents
Behind the scenes: any component needs to be added to the declarations array, so Angular knows which components we have.
Angular will then create a component when it finds it in one of two places:
either in a template, or in the routes.
One place that doesn't work by default is when we create a component manually with code. Instead we need to let Angular know it will need to create the Alert Component.
We add a property to the object we pass to ngModule (in the app.module file).
entryComponents, which is also an array.
It's an array of component types that will need to eventually be created without a selector or the route config.
entryComponents: [
    AlertComponent
]

In order to display the right message and then be able to dismiss the alert, we'll need a few more things.

# Data Binding and Event binding
We can't use the square brackets, etc, because we're creating the component in code - not using a selector in the template.
We have to store the component here in a variable to be able to work with it.
const componentRef = hostViewContainerRef.createComponent(alertCmpFactory);

Then we can use the componentRef. It has an instance property which gives us access to the concrete instance of the component that was created here.
This instance should have the properties we added to our component. If we type the dot, we can see message and close pop up as options. Then we can set the message equal to the message we're receiving.
componentRef.instance.message = message;

Then we'll need to manually listen to our close event. We'll have to manually store the subscription (we create a property in the component) and then call this.closeSub.unsubscribe(); and also clear the view container ref.


this.closeSub = componentRef.instance.close.subscribe(() => {
 this.closeSub.unsubscribe();
 hostViewContainerRef.clear();
});

If the app component is removed, we also want to clear that subscription, so we use onDestroy.
