# intro
Beacuse we're building single page applications, we need to handle forms through Angular. Then if we want to submit something to a server, we have to reach out via Angular's http service (which we'll cover later in the course).

Angular will parse the info given by the user and also check validation. We'll need an object representation in our TS code of the user's inputs. 
And Angular gives us this - a JS object representation of our form, which makes it simple to retrieve user values, see the state of the form, and work with it.

# Template-driven vs. Reactive approach
Angular offers two approaches
Template-driven: set up the form in the template, in the HTML code, and Angular will automatically infer the structure of the form, which controls the form has, which inputs, etc, and that makes it easier to get started quickly.
Angular infers the form object from the DOM.

Reactive approach: We define the structure of the form in TS code, also set up the html code, and manually connect it. This gives us greater control.
The form is created programmatically and synchronized with the DOM.

# An example form
We don't put an action or method attribute on the fom tag in the html because we don't want an http request to be the result of submitting this form. We don't want it sent to a server - we want Angular to handle the form.

# creating the form and registering the controls
We need to make sure, in the app module, to add FormsModule to the imports array and import it at the top of the file from '@angular/forms';
With this imported, Angular will automatically create a form for us (a JS representation of the form) when it detects a form element in the HTML code. 
We can think of the form element serving as a selector for an Angular directive which then creates a JS representation of the form. We don't immediately see the JS object. If we could it would be empty because Angular does not automatically detect the inputs in our form. We might not want to add all of these elements as controls in our form. (Controls, meaning what is in the JS object). We still need to register controls manually and let Angular know which elements should be controls in our form. 

Telling Angular what our form looks like:
In the template-driven approach, we choose which inputs we want to add as contols and add the attribute ngModel.
(This is the same directive we used in two-way databinding - however, here, we'll add it without any parentheses or square brackets.)
In order for this to work and have this input recognized as a control in our form, we need to give Angular another piece of info: the name of this control.
We do this by adding the normal HTML attribute: name
<input
    type="text"
    id="username"
    class="form-control"
    ngModel
    name="username">

Now this control will be registered in the JS representation of the form. 
We set up the select the same way - it's just another type of input. 

# submitting and using the form
We want to actually see what the user entered. 
In the app.component.ts we'll add a method, onSubmit(). This should be triggered whenever the form is submitted by the user. And we'll want to output whatever the user entered.
We first need to call the method. So back in the template, instead of doing a click listener on the button of the form, because the default html behavior would be triggered. And it will also trigger a JS event, the submit event. (The button does need to be of type="submit".)
Angular takes advantage of this and gives us a directive we can place on the form element (as a whole). It's called ngSubmit and gives us one event we can listen to.
<form (ngSubmit)="onSubmit()">...</form>
This ngSubmit will be fired whenever the form is submitted.

In order to see the values that were submitted, to see the form object.
In the template-driven approach we need to set this up in the template.
In the form object/element we want to get access to the form created by Angular. 

We can place a local reference on the HTML form element: #f and pass f as an argument to the onSubmit method. 
<form (ngSubmit)="onSubmit(f)" #f>

In the onSubmit method in the TS file, we pass form of type HTMLFormElement, and then we can output the form.
onSubmit(form: HTMLFormElement) {
    console.log(form);
}

In order to get access to the form object, we need to set the local reference (in this case #f) equal to ngForm.
<form (ngSubmit)="onSubmit(f)" #f="ngForm">
This asks Angular to give us access to the form object it created automatically.

And then we pass it to the onSubmit method. It will no longer be of type HTMLFormElement, but it will be of type NgForm.
(in the TS file we need to import NgForm from '@angular/forms').
  onSubmit(form: NgForm) {
    console.log(form);
  }

Then consol.logging form, we see the form object with a value property. NgForm, then form, then value. There we'll see the key/value pairs.

# Understanding form state
The JS object also has a lot of other built-in properties. It allows us to really understand the state of our form.
We can see the controls we registered, and each control is of type FormControl (a type made available by Angular).
Each control has properties also. dirty, disabled, enabled, errors... etc.
dirty means we changed something about the form
touched means we clicked into the field(s)

These properties can all be helpful in changing the user experience.

# accessing the form with @ViewChild
@ViewChild allows us to access a local reference in our TS code. Here we have a local reference that points to the ngForm object (rather than an element); so we can use @ViewChild here (rather than passing the form to the onSubmit function).

In the app component TS file we'll use @ViewChild() in the class. We need to import it from '@angular/core'

Then we want to get access to the element which has the local reference f on it. So we pass f as a string, as an argument, to the @ViewChild(). Then we could store it in a variable named signupForm which will be of type NgForm. 
@ViewChild('f') signupForm: NgForm;

Then in the onSubmit function, we can output the form:
console.log(this.signupForm);

This gives us access to the form without passing it to onSubmit. Which can be useful for times when we need to access the form not just at the time we submit it. 

# adding validation to check user input
We should always still have validation on the server; but we can enhance the user experience by also having it in our application on the frontend. We can check that none of the inputs are empty and that the email address is a valid email address.

Since here we're using the template-driven approach we can only add the validation in the template.
We can add the required attribute to the inputs.
<input type="text".... required>
required is a default HTML attribute, but Angular will detect it and it will act as a selector for a built-in directive shippig with Angular and it will automatically configure the form to take required into account.

On the email input we can add required and email. 
email is not a built-in HTML attribute, but it is still a directive made available by Angular.
(A list of all available validators will be in a later lecture)

Angular will track this on a control level and on a form level. It will add some classes to the element: ng-dirty ng-touched, ng-valid, etc. Angular dynamically adds/removes these CSS classes, giving us information about the state of the individual control. With that information we can style these inputs conditionally.

# built-in validators and HTML5 Validation
https://angular.io/api/forms/Validators
https://angular.io/api?type=directive
(in the above directive list search for validator)

We can enable HTML5 validation (by default Angular disables it):
add the ngNativeValidate to a control in the template

# using the form state
We can disable the submit button if the form is not valid:
We'll add some property binding - we'll bind to the disabled property which will set the disabled state to true or false depending on a certain condtion, in this case, whether the form is valid.
[disabled]="!f.valid">Submit</button>
We get access to the form with the local reference f on the form element. 

In the stylesheet of the app component, we can add styles for the different classes Angular adds. 
.ng-invalid {
  border: 1px solid red;
}
However, the above code will add a red border to each input plus the overall form. Angular adds the class to each input and to the form element. And it will default to this even on a firt load/render.

To make it not target the whole form or groups of inputs, we could do the following instead:
input.ng-invalid {
  border: 1px solid red;
}

To make sure the warning doesn't show up til the user clicks in the input and then away, we can add the ng-touched class:
input.ng-invalid.ng-touched {
  border: 1px solid red;
}

We can also add a warning message and add a conditional render:
<p *ngIf="">Please enter a valid value.</p>

# outputting validation error messages
We can use the bootstrap class help-block and then only show the message if the input is not valid.
<span class="help-block">Please enter a valid email.</span>

A way to get access to the control created by Angular is by adding a local reference to the input element and setting it equal to ngModel.
<input type="text"... #email="ngModel">
So just like the form directive automatically added by Angular when it detects a form element, the ngModel directive exposes some additional information about the control it creates.
Then we say we want to attach the span if email is not valid (but has been clicked into by the user).
<span class="help-block" *ngIf="!email.valid && email.touched">Please enter a valid email.</span>

# set default values with ngModel property binding
