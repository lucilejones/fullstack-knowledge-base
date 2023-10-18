# what is an observable?
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

