# what is an observable?
And observable is an interface used to handle various asynchronous or event operations. Observables provide support for passing messages between different sections of the app. Observables are a stram of data that we subscribe to receive info when the data changes.

An observable can be thought of as a data source.
In our Angular project, an observable is an object we import from a third-party package, rxjs. The observable here is implemented in a way that it follows the observable pattern, so we have an observable and we have an observer. In between, we have a stream - a timeline. On the timeline we can have multiple events emitted by the observable or the data package. (Depending on the data source of that observable.)

Various data sources:
(user input) events, http requests, triggered in code, ...

The observer is our code. It's the subscribe funtion we've used before (or has something to do with it). We have three ways of handling data packages - we can handle the regular data, we can handle errors, or we can handle the completion of the observable. These are the three types of data packages we can receive. And in these hooks, our code gets executed. We can determine what should happen if we receive a new data package, if we receive an error, or when the observable completes. (Side note, an observable doesn't have to complete.)

This is how the observable pattern works and we use it to handle asynchronous tasks - for when we don't know when something will happen or how long it will take. 
We have used callbacks or promises, but observables is just a different approach for handling these. And Angular uses observables a lot. 
Observables have one major advantage - their operators.


# Analyzing Angular Observables
params is an observable to which we subscribe (params that we learned about and used with all the routing lessons) - to be informed about changes in data.
params is the observable, a stream of route parameters which gives us a new route parameter whenever we go to a new page, whenever the parameter in the URL changes. And then in the function (in our user component) that we pass to subscribe, we get the new params and we can extract out teh relevant param (in this case, the id).

That's how this built-in observable works.

We can also create our own observable.

# The Core of observables
Observables are not baked into JS or TS. They're added by a package rxjs. 

In the home component, in the ngOnInit we can build a new observable. 
We can import {} from rxjs. There are several ways to create a new observable.
Here, we'll import the interval method, and then in the ngOnInit we can call interval and pass in a number. This is the amount of time (usually in miliseconds) for how often an event will be emitted.
This gives us an observable and then we can subscribe to the observable.
Then we pass an anonymous function as the first argument; it's the handler for all the data values that are emitted. It gets the value that is emitted as an argument (here we name this argument count because it will be an incrementing number - for interval). Later we'll build an observable from scratch and we'll control the emitted values. 
interval() will fire a new value every second and inside the function we'll just console.log(count)

  ngOnInit(): void {
    interval(1000).subscribe( count => {
      console.log(count);
    })
  }

When we open the browser and open devtools we can see in the console that an incremented number is logged every second. Even if we navigate away (to user1 or user2), the count keeps on going. 

There are certain observables that emit a value once and are done; for example, an http request where we get back a response. But other observables keep emitting values. To stop that, and to prevent memory leaks, we need to unsubscribe from observables we're no longer interested in. 

If we navigate back to the home page, it starts a new observable count back at 0, but also keeps the other one going. If this happens behind the scenes, we quickly run out of resources, slow down the app, and introduce a memory leak because our memeory gets occupied a lot by data we don't need. 

In order to be able to clear an observable, we need to store it. So in the same home component we'll name a private property named firstObsSubscription and it will be of type Subscription (which needs to be imported from rxjs). 

subscribe() actually returns such a subscription, so when we subscribe we can store that subscription in our firstObsSubscription propterty. 
    private firstObsSubscription: Subscription;

  ngOnInit(): void {
    this.firstObsSubscription = interval(1000).subscribe( count => {
      console.log(count);
    })
  }

So we're not storing the observable; we're storing whatever the subscribe() returns, and it returns a subscription. 
So then we can implement onDestroy (which needs to be imported from '@angular/core'), and that forces us to use the ngOnDestroy lifecycle hook. And inside ngOnDestroy we can use our subscription and call unsubscribe();
That means whenever we leave that component we clear that subscription. 

Common question: why don't we need to unsubscribe from params? In the user component. 
The answer is that Angular does it for us. Any observable that is provided by a feature of Angular - all the Angular observables are managed by Angular. 

# Building a custom observable
In the ngOnInit() in the home component, we can comment out the interval provided by rxjs, and instead build it manually.
We can store our own observable in a constant and name it customIntervalObservable. 
To create a new Observable, we need to import Observable (the type itself) from rxjs. 
We call a create() method on Observable. 
create() takes a function, so we'll pass in an anonymous arrow function, and rxjs will pass in an argument automatically. That argument is an observer.
The observer is the part that is interested in being informed about new data, errors, or the observabsle being completed. 
In the function, we can use the setInterval method. Then we can use observer.next() to emit a new value. observer has a few important methods: next(), error(), complete()
To have an incrementing number here, we can introduce a variable, count, which starts at zero. 
Inside our setIterval we pass count into next() and increment count by 1. 
  const customIntervalObservable = Observable.create(observer => {
    let count = 0;
    setInterval(()=> {
      observer.next(count);
      count++;
    }, 1000);
  });

Then we also subscribe to our observable. And we pass a funtion that accepts the data we're emitting. We could name it data. 
    customIntervalObservable.subscribe(data => {
      console.log(data);
    });

We also have to store the subscription in the subscription variable.
this.firstObsSubscription = customIntervalSubscription.subscribe(data => {
  console.log(data);
});

# errors and completion
The counter we built can't fail, but especially when working with http request we need to think about error handling.
In our example we can fake an error. And create a new error object. 
if (count > 3) {
  observer.error(new Error('Count is greater than 3!'))
}
It will throw an error and then cancel - the observable will stop. 

In order to handle an error, we pass another argument to the subscribe(). This will be the function we want to run when there is an error. We;ll get the error as an argument. The simplest thing would be to console.log the error, but we could send it to our own backend, give the user an error message, etc. 
    this.firstObsSubscription = customIntervalObservable.subscribe(data => {
      console.log(data);
    }, error => {
      console.log(error);
      alert(error.message);
    });

Completing an observable - can be part of the normal process. An http request will complete when we receive a response.
We can also complete an observable manually. 
if ( count === 2) {
  observer.complete();
}
In this we don't need to pass any arguments. 
When we call complete() the observable comes to a halt.
If we want to react to the completion, we can add a third argument to the subscribe method, and that is the completion handler function. It gets no arguments; it's just a function that can do some clean-up work or other things we need to do. 
    this.firstObsSubscription = customIntervalObservable.subscribe(data => {
      console.log(data);
    }, error => {
      console.log(error);
      alert(error.message);
    }, () => {
      console.log('Completed!')
    });

We don't need to unsubscribe if the observable completed. 

The completet function (that we pass to .subscribe() ) will not run if we get an error. An error cancels the observable; it does not complete it. 

Whenever we set up our handler functions for the subscribe(), rxjs merges them together into one object, and passes that object - the observer - to the observable. And inside of the observable it will interact with the observer and let the observer know about new data, errors, etc. 

We will rarely build our own observables. More often we'll use observables that come with libraries, like Angular. (Like the route.params observable we've already used.) 

In general, observables wrap an event source and give us data, errors, or the complete event. We will subscribe and pass in functions that deal with that data or those errors. 

# Understanding operators
Operators are a feature of observables.
Sometimes we don't want to use the raw data we get from an observable; we might want to transform it or filter out certain data points. We could do this inside the subscription, or in the function we pass to the subscription, but there is a more elegant way.
We can use built-in operators in between the observable and the subscription. That means the data points first reach these operators that do something to the data (there are a lot of built-in operators), and then we subscribe to the result of these operators. 

We can use operators on any observable to change the data before we subscribe to it. 
We call the pipe() method. The pipe method is built into rxjs. 
Then we import from 'rxjs/operators';
We can import the map operator.
import { map } from 'rxjs/operators';
We call map() as a function inside of pipe. map() takes a function as an argument.
That function gets the data as an argument. That is the data we would otherwise get here in the first subscribed callback (the first function we pass to subscribe). The current data that's being emitted by the observable. 

Then we need to change our subscription. The pipe doesn't change the observable. So if we still subscribe to the observable, it'll be the same data.

So then we take that whole piece of logic and subscribe to it instead.

This will be very useful when we're fetching data from a complex web server and want to transform that data before we use it in a component. 

The pipe() method takes an unlimited amount of arguments (if we have more than one operator), separated by a comma. Each argument would be an argument imported from 'rxjs/operators'; They will execute after each other. 
(Can look up rxjs operators.)

filter() is another operator and has to return true or false. We can give it a condition and if that is true for the data point it will return true and move on to the next operator. 

Operators allow us to build up a chain of steps we want to filter our data through.

# Subjects
With a service and an event emitter.
There is a better, more recommended, way than using an event emitter. 

Subjects are unique observables that act as both the 'observer' and the 'observable'. Subjects allow us to emit new valus to the subscription stream using the next() method.

A Subject is something we import from 'rxjs' instead of EventEmitter.
(In this example, in the user.service.ts file)

The Subject is also a generic type where we define the data which we eventually be emitted (in this case a boolean).
activatedEmitter = new Subject<boolean>();

Then in the user component instead of calling emit() we call next().
Because a Subject is a special kind of observable: it is also an object we can subscribe to, but it is active rather than passive; we can call next() from outside the observable and it can be triggered from our code.
(A regular observable wraps a callback, event, etc and can have next() called from inside the observable when we create it.)
This more active observable is perfect for when we want to use it an an event emitter (when we don't have a passive event source, like an http request or DOM event), when we have something we want to be actively be triggered by us in our application. 

In the app component we still subscribe to the activatedEmitter (because it is still an observable).

Subjects are better than event emitters because they are more efficient behind the scenes and we can also use operators on them because they're observables. 

We should also unsubscribe to Subjects when we no longer need them.

We add OnDestroy to the app component (or any component where we subscribe to the observable). We need to stor the subscription with private activatedSub: Subscription (import Subscription from 'rxjs')

Where we call subscribe, we set it equal to this.activatedSub, so the subscription is stored there, and then we can unsubscribe (in ngOnDestroy).

We only use Subjects as replacements for EventEmitters if we're using them as cross component event emitters (to communicate across components with services), where we manually call next() (or when we were using emit()). We don't use Subjects when we're using @Output. We still use the Angular EventEmitter.


Official docs: https://rxjs-dev.firebaseapp.com/
