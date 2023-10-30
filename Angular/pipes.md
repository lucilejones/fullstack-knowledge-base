# what are pipes?
Pipes are simple functions we can use in HTML template expressions that accept and initial value and return a transformed value.

Pipes are a feature built into Angular 2 which allow us to transform output in our template. 
There are different pipes for different types of output, and also synchronous and asynchronous data. The general idea is always the same.
For example, we might have a username that we want to display in all uppercase. We don't want to actually change the property to be all uppercase, but we can use the uppercase pipe to transform the data when it gets displayed. (uppercase is a built-in pipe; we can also build our own pipes).
<p>{{ username | uppercase }}</p>

# Using pipes
In our template (since a pipe is responsible for transforming the output - the template is logically the place to use pipes):
To change the server.instanceType to uppercase, we just add the pipe symbol | and then the name of the pipe. 
{{ server.instanceType | uppercase }}

The date: we can use the date pipe (another built-in pipe).
{{ server.started | date }}
But this might not be exactly what we want it to look like, so we can also configure pipes. 

# parametizing pipes
We pass a parameter to it (it has to be a pipe that can take a parameter). We can add a colon and then a string. The date pipe can take a parameter and expects a string.
{{ server.started | date: 'fullDate' }}
This will make the output Monday, August 9, 1920
If we want multiple parameters, we separate them with colons.

# where to learn more about pipes
angular.io
click on API reference
then enter pipe as a filter
click on the pipe we want to use and there's more details for how to use with an example

# chaining multiple pipes
To use multiple pipes we can type another pipe symbol and then put another pipe. The order might be important. The list of pipes will generally be parsed from left to right. 
{{ server.started | date: 'fullDate' | uppercase }}
In the above example, the date pipe gets applied to the server.started value, and then the uppercase pipe gets applied to the result of that operation.
If we try to do the uppercase pipe and then the date pipe we'll get an error. It isn't able to uppercase the originally date (server.started) because it isn't a string.

# creating a custom pipe
We create a new file with a descriptive name.
shorten.pipe.ts
We're going to create a pipe to shorten a server name.

This file should hold a class, and we'll name it ShortenPipe.
export class ShortenPipe {}

It needs to have one special method to be used as a pipe.
We can implement (and import) the PipeTransform.
export class ShortenPipe implements PipeTransform {

}

Then we need to add the transform method. It needs to receive the value that should get transformed. (It can also take arguments.)
    transform(value: any) {
        
    }

To shorten this value, we return (transform always needs to return something) a new string which should be the old value and use the substring method and define how long we want the string to be. (We start with index 0 and then how many characters we want.)
return value.substr(0, 10);

Then to use this pipe, we need to go to app.module and add it in the declarations array. We also need to import it with the file path. 

We need to add the @Pipe decorator. We specify the name.
@Pipe({
    name: 'shorten'
})

Then we can use it in our component. In the template, on the server.name, we add a pipe symbol and the name of our pipe.
{{ server.name | shorten }}

We can add a new string to the end of the shortened string. Three dots to show the while title is not being displayed. 
return value.substr(0, 10) + ' ...';
We also might want to check if the original title is longer than 10 characters and only apply the shortened value and display the three dots if it's shortened. 
    transform(value: any) {
        if (value.length > 10) {
            return value.substr(0, 10) + ' ...';
        }
        return value;
    }

# parametizing a custom pipe
It could be nice to allow the user to specify the number of characters to display.

We can have the transform method receive a second argument, maybe limit of type number.
transform(value: any, limit: number) {}

Then we use limit in the check and use limit in the substring method. (Instead of hard-coding in 10.)

If a parameter isn't passed, it'll always fail the check and just return the original value.

We can pass a parameter to our pipe in the HTML:
{{ server.name | shorten:15 }}

We can also chain our pipe with other built-in pipes. Or add multiple parameters to our pipe.

# example: creating a filter pipe
If we want the user to be able to filter the list of servers by status.
In the HTML file we create a new input element and use two-way databinding.
<input type="text" [(ngModel)]="filteredStatus">
Then we set up a new property in the component (in the TS file).
We'll add filteredStatus as a property beneath our list of servers and set it as an empty string.
filteredStatus = '';

We can use the CLI to create a new pipe file.
ng g p <pipename>
example: ng g p filter will generate filter.pipe.ts and will prepopulate the file with the @Pipe({}) decorator and the imports and the transform() method.

In the transform, the first argument (after the value) will be what the user entered. Here we'll call it filterString of type string. 
Then we want to only return elements of the array that fulfill this filter string. Where the status of the server matches the filter string.

We'll first check if value.length (and here, value will be the array of servers) is zero, if it's empty. Then we'd just return value. 
if (value.length === 0) {
    return value;
}
Otherwise we want to loop through all the items in the value and check if the status of each server matches the filter string.
for (const item of value) {
    if (item.status)
}
Or we can pass the to be filtered property as a second argument to the transform function.
Then we need a result Array and we'll push that item to the result array if its status matches the filter string.
  transform(value: any, filterString: string, propName: string): any {
    if (value.length === 0) {
      return value;
    }
    const resultArray = [];
    for (const item of value) {
      if (item[propName] === filterString) {
        resultArray.push(item);
      }
    }
    return resultArray;
  }

Then in the app component we apply it in the ngFor loop.
We can add the pipe to the ngFor because we're transforming the output, and an ngFor loop is simply part of the output.
*ngFor="let server of servers | filter:filteredStatus:'status'"

We add the pipe symbol with our pipe name, a colon, the first parameter (filteredStatus - the property that holds the string we want to filter by; in the property binding on the input element), and the second parameter (the string 'status' - the propName; we want to filter on the status property of each server).

Then no servers will be displayed by default, but then we can type a status into the input.
If we want to see all the filters we can adjust our condition to also just return the value if the filter string is an empty string.

# pure and impure pipes (how to "fix" the filter pipe)
If we allow the user to add a new server:
Add a button to the template with Add Server.
<button class="btn btn-primary" (click)="onAddServer()">Add server</button>

Then in the ts file we add the onAddServer() method (with just a hard-coded server).
  onAddServer() {
    this.servers.push({
      instanceType: 'small',
      name: 'New Server',
      status: 'stable',
      started: new Date(15, 1, 2017)
    })
  }

If there is text in the input field and we click on the Add Server button, the new server does not show up in the list. The reason is Angular is not rerunning the pipe on this data whenever it changes. (To be precise: updating arrays or objects doesn't trigger it.) Otherwise, Angular would have to rerun this pipe whenever any data on the page changes - that would cost a lot of performance. 
We can force it to work, but keep in mind that whenever we update data on the page the pipe will update. This might lead to performance issues, so we need to keep that in mind.
To do this, we add a second property to the @Pipe({}) decorator, pure and set it to false (the default is true).
@Pipe({
  name: 'filter',
  pure: false
})

# understanding the "async" pipe
This built-in pipe helps with handling asynchronous data.
We can have another property that holds our appStatus and should load after 2 seconds (we'll just simulate that here in this example). We can set the property equal to a promise.
appStatus = new Promise();
(We can imagine this data being return from an http call, etc).

We'll initialize the Promise with a callback method passed to the constructor where this method takes two arguments, resolve and reject, the two functions we can execute inside of the Promise. And in the Promise, in the callback function, we'll use setTimeout and then another method will get executed. It will resolve the appStatus. 
  appStatus = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('stable');
    }, 2000);
  });

This will set the appStatus to stable, but only after 2 seconds.
So if we try to output the appStatus in the template with just string interpolation, we'll see App Status: [object object].
<h2>App Status: {{ appStatus }}</h2>

This makes sense, it's a promise and a promise is an object. After 2 seconds, though, it will be a string. We know that but Angular doesn't know. It doesn't watch the object. 

There's a pipe we can use to make the transformation of this data easier. 
We know it will resolve to a string after 2 seconds, so we can add the pipe symbol and the name async.
<h2>App Status: {{ appStatus | async }}</h2>
This is a built-in pipe.
Then when we first load the page the App Status: is blank and then it shows stable after 2 seconds.
The async pipe recognizes that this is a promise (and as a side note, this will also work with observables, it will subscribe automatically), and after 2 seconds it will recognize that something changed, the promise resolved (or in the case of an observable, that data was sent through the subscription), and it will print this data to the screen.
We'll see this async pipe being used in the http section later in the course.
