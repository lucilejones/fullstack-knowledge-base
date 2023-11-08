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
