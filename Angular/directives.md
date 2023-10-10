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

# Using HostListener to Listen to Host Events
We can react to some events occuring on the element the directive sits on. 
Inside of the BetterHighlightDirective we add a new decorator, the @HostListener, which needs to be imported from '@angular/core', and add it to some method we want to execute:
@HostListener() mouseover() {}
Then that can be triggered whenever some event occurs, which can be specified in the parentheses, as an argument, as a string. 
@HostListener('mouseenter') mouseover() {}
mouseenter is one of the events supported by the DOM element this directive sits on. 
In the mouseover() we received the eventData of type Event.
We can also listen to custom events here. Just like a method we exectue when we add a click listener, or whatever the event is and then pass the method between quotation marks.
We want to change the background color of the element. So we'll set the style on mouseenter.
Then we can add another @HostListener for mouseleave.

# Using HostBinding to bind to Host properties
We have another decorator that we can use in place of the renderer: @HostBinding, which also needs to be imported from '@angular/core'
We need to bind this to a property who's value will become important. In this example, we'll use backgroundColor, a new property we create here, of type string.
@HostBinding() backgroundColor: string;
We can pass a string defining to which property of the hosting element we want to bind.
Properties of the hosting element, that is what we also access in the BasicHighlightDirective. Style, backgroundColor would be such a property.

@HostBinding('style.backgroundColor') backgroundColor: string;
We're accessing the DOM property, so the camelCase is important. 

With this, we're telling Angular, on the element where this directive sits please access the style property and then the subproperty backgroundColor and we set that equal to whatever backgroundColor is set here. We can add that to the mouseenter HostListener. 
this.backgroundColor = 'blue';
[we always need to make sure we're accessing a property the element has; for example, only inputs have a value property]
We'll also want to set an initial color so we don't get an error before we mouseover the first time. 
@HostBinding('style.backgroundColor') backgroundColor: string = 'transparent';

We can bind to any property of the element we're sitting on.

# Binding to directive properties
We can use custom property binding. 

In the better-highlight directive we can add an @Input called default color (and we need to import Input from '@angular/core'), and one for the highlight color.
@Input() defaultColor: string = 'transparent';
@Input() highlightColor: string = 'blue';
We have some default values to use, but they can be overwritten from outside. 
Then we change the @HostBinding backgroundColor to the this.defaultColor (instead of hard-coding blue in here.)
Then in the mouseover we change it to this.highlightColor.
And then defaultColor again once we mouseleave.

To bind it from outside:
In our app component where we use the better-directive, we can bind to defaultColor, and maybe set it to yellow. And also bind to highlightColor, which could be red.
We actually need to set the this.backgroundColor = this.defaultColor in the ngOnInit so it starts out that way when the page is first loaded. 

# What happens behind the scenes on structural directives
The star indicates to Angular that it is a structural directive. 
Behind the scenes Angular will transform them into something else, it will transform an ngIf usage (for example) into something where we end up with the tools of property binding and so on. 
We could rewrite the same code with an ng-template instead.
Inside the ng-template we have the content we conditionally want to render. 
ng-template is an element that is not rendered itself, but allows us to define a template for Angular to use once it determines that this element needs to be rendered because the condition is true.
We add the ngIf to the ng-template, without the star, because this is the form it will get transformed to when we use the star, but instead with square brackets for property binding. <ng-template [ngIf]="!onlyOdd">...</ng-template>

<div *ngIf="!onlyOdd">
    <li>...</li>
</div>

gets transformed to


<ng-template [ngIf]="!onlyOdd">
    <div>
        <li>...</li>
    </div>
</ng-template>

# Building a structural directive
We'll create a directive called unless; and this directive will attach something only if the condition is false. 
In the directive file we need to get the condition as an input. 
@Input() and import from '@angular/core'
We want to bind to a property unless, and execute a method whenever the condition changes. 
To do that we implement a setter with the set keyword. That will turn unless into a method, though technically it's still a property, it's just a setter of the property which is a method that gets executed whenever the property changes. In this case, whenever the condition that we pass changes, or some parameter of this condition.
So unless needs to receive the value as an input, in this case of type boolean.
@Input() set unless(condition: boolean) {}
Then we check if the condition is not true (and if that is the case display something).

Our unless directive will sit on an ng-template because that is what it is transformed to by Angular, if we use the star.
So we need to get access to the template and the place in the DOM where we want to render it. Both can be injected.
In the constructor, we inject the template with private templateRef with the type TemplateRef.
Just like ElementRef gives us access to the element the directive was on, template ref does the same for a template. It is a generic type, so we can pass any.
We also need to import TemplateRef from '@angular/core'
The second piece of information we need is the view container, where we should render it.
We can name it vcRef for view container reference, which will be of type ViewContainerRef (also imported from '@angular/core').
constructor(private templateRef: TemplateRef<any>, private vcRef: ViewContainerRef) { }

We can use the vcRef whenever the condition changes to call the createEmbeddedView() method, which creates a view in this view container. And the view is our template ref. 
if (!condition) {
    this.vcRef.createEmbeddedView(this.templateRef);
} 
This template we created there is exactly this reference to the template there.
In the else we call the clear method to remove everything from this place in the DOM.

We need to make sure the directive is added to teh app.module.ts in the declarations array. 
Then in the app component we can use our directive. We still have to use the star because it still is a structural directive.

<div *appUnless="onlyOdd">
    <li>...</li>
</div>

We'll get an error that we can bind to appUnless because it isn't a known property of <div>. We have to make sure the property name shares the name of the directive, appUnless.

# understanding ngSwitch
ngSwitch is a built-in structural directive
Maybe we have a value property in our app component.
Then we have a place in our app where the value changes and we get a couple different messages we want to display for each of these values.
Then we bind to ngSwitch with property binding: <div [ngSwitch]="value"></div>
We bind to value - this is our condition, what we want to check.
Then Switch has a couple of cases we can now cover.
<div [ngSwitch]="value">
    <p>Value is 5</p>
    <p>Value is 10</p>
    <p>Value is default</p>
</div>
Then we need to add something to determine which paragraph gets shown. We only want to display one of these at at time. 
We use *ngSwitchCase and pass the value as an argument.
For the default we use ngSwitchDefault.
<div [ngSwitch]="value">
    <p *ngSwitchCase="5">Value is 5</p>
    <p *ngSwitchCase="10">Value is 10</p>
    <p *ngSwitchDefault>Value is Default</p>
</div>

We need to use the star for Angular to transform this behind the scenes. 
Can be useful if we're creating lots of ngIf conditions. 