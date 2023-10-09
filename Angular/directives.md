# Directives
attribute directives - sit on elements, just like attributes; change properties of the DOM
-look like a normal HTML attribute, possibly with databinding or event binding
-only affect/ change the element they're added to

structural directives - they also change the structure of the DOM; potentially remove a whole element
-look like a normal HTML attribute, but have a leading * (for desugaring)
-affect a whole aread in the DOM (elements get added/ removed)

# ngFor and ngIf recap
*ngFor="let number of numbers"
{{ number }}

We can't have more than one structural directive on an element. 
*ngIf="onlyOdd"
*ngIf="!onlyOdd"

# ngClass and ngStyle recap
.odd {
    color: red;
}

[ngClass]="{odd: odd % 2 !== 0}"
[ngStyle]="{backgroundColor: odd % 2 !== 0 ? 'yellow' : 'transparent' }"

# Creating a basic attribute directive
In our app folder, we create a new folder: basic-highlight
Then in that folder we create a new file: basic-highlight.directive.ts

In this file we export a class:
export class BasicHightlightDirective {}

To make it a directive, we have to add @Directive (and import it from '@angular/core') and we have to pass an object to configure it. And the one thing it absolutely needs is a selector. We place directives in our template to attach them to elements, so we need a way to give Angular that instruction - we use the selector. It should also be a unique selector, using camelCase, and wrapped in square brackets, to use attribute style.
@Directive({
    selector: '[appBasicHighlight]'
})

To check if this is working, we'll change the background color of the element where we attach this directive.
We'll need to get access to the element the directive sits on. To do this, we can inject the element the directive sits on into this directive. 
[we'll look more at injection in the next section about services - basically a way to get access to some other classes without having to instantiate them on our own.]
We inject by adding the constructor. We don't need to add anything into the body of the constructor for now, but we do want to add a couple arguments. We want to list the arguments we want to get whenever an instance of this class is created. 
Angualr is responsible for creating these instances, so if we tell it to please give us a specific type of argument (this is what injection is), Angular will try to create the thing we need and give it to us. 
In this case we need a reference to the element the directive was placed on. 
elementRef: ElementRef (the name can be whatever we want, but the type has to be ElementRef.)
constructor(elementRef: ElementRef){}
[we also used ElementRef with @ViewChild].
And we need to import ElementRef from '@angular/core'
To be able to use this data everywhere in the class, we can use the TS shortcut, private, which will make this a property of the class, elementRef, and automatically assign this value, this instance we're getting, to this property. 
With that, we get acces to the element, so we can use it now in the constructor, access the native Element and then do something with it.
elementRef.nativeElement
Though a better place is in the OnInit().
So we add implements OnInit to the class.
Just like the component, the directive also has the ngOnInit lifecycle hook. 
Then in the class we add ngOnInit() {
    this.elementRef.nativeElement.style.backgroundColor = 'green'
}
Then we can access the elementRef (which we got by using private), the native Element and the style. 

Then to use the directive we have to do two things:
We have to inform Angular that we have a new directive. 
Just like with components, Angular doesn't scan all our files, so it doesn't know.
So in the app.module.ts folder we do to the declartions array and add our BasicHighlightDirective.
It will also need to be imported from the basic-highlight folder.
Then we can use the directive in our HTML file.
In a new p tag, we add the appBasicHighlight, our own selector. We don't need to use square brackets because the directive name is just the selector we set up in the directive file. The square brackets aren't part of the name, they're just part of the selector style telling Angular to please select it as an attribute on an element. So we add it just like an attribute of the paragraph. 

# Using the renderer to build a better attribute directive
(the above way is not the best way of changing the style)
Angular is able to render our templates without a DOM and then the property would not be available. It's not a good practice to directly access our elements. 

We'll create a new directive. 
Using the CLI: ng g d better-highlight
We can also put it into its own folder. better-highlight (and make sure to update the folder bath for the app.module.ts file)
We could also create a central shared folder or a directives folder.

In the betterHighlightDirective file we're going to inject the better tool.
It's the renderer of the type Renderer2 (which also needs to be imported from '@angular/core').
With this injected, we can use ngOnInit in the class again.
Then in the ngOnInit(){} we can use the renderer.
ngOnInit() {
    this.renderer.setStyle(this.elementRef.nativeElement, 'background-color', 'blue')
}

We first call the property we created, the private renderer, and there we get some methods we can use to work with the DOM. In this example we'll use the setStyle() method. 
In order to use that we need to have the element that we want to set the style on. We can inject the element reference. 
We add another private property which we can name elementRef and will be of type ElementRef (which we need to import from '@angular/core').

Then we can use it in the setStyle, by calling this.elementRef and access the native element. We can't pass the reference itself; we need to get access to the underlying element and then pass it as a first argument to setStyle(). 
The next argument is which style we want to set; in this case the background color. The third argument we pass is the value we want to assign. The fourth argument is a flags object. It's optional, but we can set a couple of flags for this style, for example an Important tag (!).

Then in the app component we add the directive. 
<p appBetterHighlight>Style me with better directive.</p>
It's available through the selector because we also added it to the app module. 

The rendered is a better directive because Angular is not limited to running in the browser. It also works in these environments where we might not have access to the DOM. So if we try to change the DOM by directly accessing the native element and the style of that element (like we did with the basic directive), we might get an error. Most of the time, it might be okay, but it's a better method to use the rendered for DOM access and to use the methods it provides. 