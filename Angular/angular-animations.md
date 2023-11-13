With Angular 4+, animation functions were moved into their own package and we need to add a module to the imports[] array in the AppModule.
- We probably need to install the new animations package:
npm install --save @angular/animations
- add BroswerAnimationsModule to the imports array
Also at the top of the app module file:
- import { BroswerAnimationsModule } from '@angular/platform-browser/animations
- then we import trigger, state, style, etc from @angular/animations instead of @angular/core

Angular 2 ships with its own animation system that we can use for things that are harder to handle with normal CSS transitions or animations.

# setting up a starting project
In the example project, there are two buttons and an empty div under them. That's the div we want to animate

# Animations triggers and state
Animations are set up in the @Component decorator.
In the app.component.ts file, under the templateUrl, we add an animations array. In that array we define any animations the template should be aware of. Each animation has a trigger, and trigger needs to be imported. With trigger, which is a function, we tell Angular we want to define a certain name we can place in our DOM, in our template, which should trigger a certain animation. The second argument defines the animation this trigger should toggle. animation is of type AnimationMetadata[] and this is an array. So we pass an array as the second argument.
animations: [
    trigger('divState, [

    ])
]

We're going to be manipulating the state of the div element. 
We can add the trigger to the div (in the template). We use square brackets similar to property binding, but with the @ symbol and the name of the trigger. We need to bind it to something, to a condition; in this case we'll use a simple property named state (which doesn't exist yet - we need to create it in the component)
<div [@divState]="state"></div>

In the TS file:
state = 'normal';

So then in the trigger we define two states: normal and highlighted.
And that's how animations in Angular work - we transition from state 1 to state 2. 
We pass state into our array (which also needs to be imported and is an animation related method) and we'll have two states here.
  animations: [
    trigger('divState', [
      state('noraml'),
      state('highlighted')
    ])
  ]

Then we set up each state by giving them names. The first will be 'normal' and the other will be 'highlighted'

The second argument of the state method is what it should look like. We pass a style object/method also imported. And in that method we define the stule of the state. We pass a JS object and here it's almost like writing CSS code, but don't use any dashes. We can use quotes or camelCase.
We want to background color to be red and to set transform to translateX(0) ot have it at its default position and not move anywhere to the left or right.

In the template, we'll set the width and height of the div:
      <div
        style="width: 100px; height: 100px"
        [@divState]="state"></div>
    </div>

Back in the TS code we want to set our hightlighted state.



