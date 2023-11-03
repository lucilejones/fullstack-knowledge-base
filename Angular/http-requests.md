# http & backend interaction

# How does Angular interact with backends?
Connecting Angular to a database
We don't connect directly. We don't enter database credentials into the Angular app or anything like that. That would be highly insecure.

We send http requests to and get http responses from a server, and API. 

We'll get back data, mostly in a JSON format. On that server we can have code that does interact with a database to store and fetch data.

Other reasons to interact with a server: file upload, analytics, etc.
(We'll use a dummy backend in this course.)

# anatomy of an http request
- HTTP verb - post, get, put
- URL (API endpoint) - example: yourdomain.com/posts/1
(Use docs to know which endpoints support which operations - or if we build our own we'll setup which endpoint support which operations)
- Header (metadata) - some default headers will be appended by the browswer and by Angular, but we can also append our own headers; example: {"Content-Type": "application/json"}
- Body - core data that is attached to a request; example: { title: "new post"}

# backend (Firebase) setup
Firebase is a whole backend solution (not just a database - more than an alternative to MondoDB)
- Go to the Firebase website
- Login with Google account - choose the free plan
- Go to the console and click on Add Project / Create Project
- Give the project a name and use the default settings
- Click button to create project
- Once it's loaded there will be an interface for the project
- On the left click on database (or Realtime database)
- Click on Create Database
(there was another step in here)
- Select Start in test mode and click Enable
(later we'll add authentication)
- The URL will be the URL we can send requests to

# Sending a POST request
In the app.component.ts file we have a function, onCreatePost() which will get some data in this format:
postData: { title: string; content: string }

Let's first console.log(postData);

Then we need to unlock an Angular feature: in the app module file we need to add the HttpClientModule. We import it from '@angular/common/http'
Then we add the module to the imports array.
This unlocks the client Angular offers for our whole project.

In the app component we'll inject our HttpClient and store it in a property that we can call http. 
constructor(private http: HttpClient) {}

We also need to import HttpClient from '@angular/common/http';

With that imported we can use it to send http requests.
In the onCreatePost method:
this.http.post()

We call the post method on http. The method takes a couple arguments. First is the url we want to sent to. For this example it's the url we find in our Firebase real-time database.
(https://academind-http-requests-default-rtdb.firebaseio.com/)

On a custom or a different REST API we would have clearly defined endpoints, like /posts/add. 

For Firebase we have the starting point URL that we can add endpoints to the url and they will get replicated as folders in the database. 
This will look like we're communicating directly with a database, but we're not. We're communicating with a REST API provided by Firebase. They just translate the path we're sending it to to a folder structure in the database.

So in this example we'll add posts to the end of the url and (for Firebase) we need to add .json.

this.http.post('https://academind-http-requests-default-rtdb.firebaseio.com/posts.json')

We also need a request body, the second argument. 
this.http.post('https://academind-http-requests-default-rtdb.firebaseio.com/posts.json', postData);

The Angular HttpClient will take our JS object and convert it to json data for us. 

The way we have this setup, the request will not actually send.

Angular uses a lot of observables. Http requests are also managed by observables. If we're not subscribing to the prepared HTTP request, to the observable that wraps the HTTP request, Angular and RxJS think we're not intersted in the response, so the request doesn't even get sent.

post() will return an observable. It doesn't give us the response as a return value. It gives us an observable that wraps the request.
So in order to get access to the response we need to call subscribe.
    this.http.post(
      'https://academind-http-requests-default-rtdb.firebaseio.com/posts.json',
      postData
      ).subscribe(responseData => {
        console.log(responseData);
      });

The HttpClient will automatically extract the data attached to the response, the response body, though we also have ways of accessing the full response (that we'll learn later).
For now we'll just console.log the responseData.

We don't need to manage this subscription (like unsubscribe) because it will complete after being done and it is an observable provided by Angular that we don't need to manage subscriptions. 

For post requests, browsers always send two. The first is of type OPTIONS - this will first check is the post request is allowed to be sent, and if that gets a success response it will send the actual response. 

If we click on the Networks tab we can see two post request with lots of information. (Headers, etc) In the request payload we see the json data that was attached. 

We also see the responseData that we sent to the console.

# GETting data
We'll add a new private property to the app component ts file:
private fetchPosts() {}

We'll use the http client, but now use the .get() method.

this.http.get()
We'll use the same url we used to store the data. 
  private fetchPosts() {
    this.http.get('https://academind-http-requests-default-rtdb.firebaseio.com/posts.json')
  }

If we look at Firebase we'll see the posts node with the post we sent. It has the key which is an id automatically generated by Firebase. 
We can expand the node and the id to see our data.

GET requests have no second argument because we're not sending data, only requesting data. 
We do need to subscribe. If we don't have a subscription Angular won't actually send the request.

  private fetchPosts() {
    this.http.get('https://academind-http-requests-default-rtdb.firebaseio.com/posts.json').subscribe(posts => {
      console.log(posts);
    });
  }

Then we cann the fetchPosts() method from the onFetchPosts() {} 
  onFetchPosts() {
    this.fetchPosts();
  }

We also want to call this from ngOnit() {}
That way, whenever the page first loads we want it to fetch the posts.

If we reload the app and then look in the console, we'll see we're getting back a JS object with one key/value pair. The key is that id string from Firebase and the value is the originaly object we stored. 
-Ni76LzXehk2_6Lucipu: {content: 'This is a test!!', title: 'test'}

If we want to store all the posts in an array that we can then loop over, we would need to transform the object to an array. 
For that we'll use an observable operator.

# using RsJS operators to transform response data
We could transorm data inside the subscribe method in the fetchPosts function; however, it's better practice to use observable operators because it allows us to write cleaner code with different steps we funnel our data through that easily be changed or adjusted. 

So before we call subscibe we can call .pipe(). Pipe is a method that allows us to funnel the observable data through multiple operators before  they reach the subscibe method.
We're going to use map so we import it from 'rxjs/operators'
The map operator allows us to get data and return new data which is automatically re-wrapped in a observable so we can still subscribe to it.
.pipe(map(responseData => {

}))

So we add map as an argument to pipe; it's a function we call and that takes another function as an input which will get our responseData and should return the converted response data. Here we want to return an array of posts instead of an object with they key, etc.
We need to manually loop through all the keys and create a new array.
We create a new postsArray that is initally empty, then we'll do a for...in loop that goes through all the keys in the responseData. And then we want to push each piece of data into the postsArray. 
We push the responseData[key], accessing the key we're currently looking at in responseData - so we're accessing the nested JS object and pushing that object into the postsArray.
We want to push the object as a new object (using curly braces and spreading in the object), so we can add a new key/value pair to the object. That should be an id field that stores the key.
    .pipe(map(responseData => {
      const postsArray = [];
      for (const key in responseData) {
        postsArray.push({...responseData[key], id: key})
      }
    }))

The key is a unique id generated by Firebase. We'll want to have the id key for deleting, updating, etc.

It's good practice to wrap the for...in loop in an if statement. If the responseData has key as its own property. That way we're not trying to access the key of some prototype.

Then we need to return postsArray. So now the array will be forwarded to our subscribe function.
  private fetchPosts() {
    this.http.get('https://academind-http-requests-default-rtdb.firebaseio.com/posts.json')
    .pipe(
      map(responseData => {
        const postsArray = [];
        for (const key in responseData) {
          if (responseData.hasOwnProperty(key)) {
            postsArray.push({...responseData[key], id: key})
          }
        }
        return postsArray;
      })
    )
    .subscribe(posts => {
      console.log(posts);
    });
  }

# using types with the HttpClient
Right now the way we have the code set up, posts (in the subscribe) is of type any. That means TS doesn't know what our posts should look like.
The postsArray is also of type any. And it doesn't know the format of our responseData, that object is totally unknown to it.

So we can assign a type to the argument we're getting in the map function.
map((responseData: ) => {})
We know it will be an object with a randomly generated string (we'll use a placeholder property name for this with square brackets); this means any key string we have here. We don't know the exact property name, but we know it's a value that can be interpreted as a string.
The value held by that property is our actual postData. 

So we can create a Post model (in a post.model.ts file). With our title: string; content: string. We'll also add an id with a question mark to show that's it's optional. id?: string.
export interface Post {
    title: string;
    content: string;
    id?: string;
}

So then we import the Post into our app component ts file (both for the onCreatePost function and for the map function).

onCreatePost(postData: Post) {}
map((responseData: {[key: string]: Post}) => {})

Now TS knows what's inside of responseData, so we can set the type for the postsArray.
const postsArray: Post[] = [];

However, instead of doing it this way, Angular has a more elegant method.
the .get() method is a generic method. We can use the angle brackets to store the type that this response will actually return as a body once it's done.
this.http.get<{ [key: string]: Post }>()
(Then we can take that out of the map method)

This is also available on post requests; it's available on all requests. It's optional, but helpful. 

In the post we can add the object that we're getting back which has a name property that is set to a string.
.post<{name: string}>()

So we can define the response data types on these generic HTTP verb methods.

# outputting posts
Now, instead of console.logging the posts, let's output the data.
Since we've transformed the data into an array of posts, we can set this.loadedPosts to the data we get back.

The loadedPosts property we can set to type Post[]
loadedPosts: Post[] = [];

this.loadedPosts = posts;

Then we need to use loadedPosts in our template.

We'll make the "no posts available" paragraph conditional and then create an element to display the posts.

<p *ngIf="loadedPosts.length < 1">No posts available!</p>
      <ul class="list-group" *ngIf="loadedPosts.length >= 1">
        <li class="list-group-item" *ngFor="let post of loadedPosts">
          <h3>{{ post.title }}</h3>
          <p>{{ post.content }}</p>
        </li>
      </ul>

# showing a loading indicator
We'll add a new property to the ts file, isFetching and we'll default set it to false. And we'll set it to true whenever we start fetching posts.
In the fetchPosts function:
this.isFetching = true;

Then, once we're done, in the subscribe function we can set isFetching back to false.
We also need to add !isFetching to the condition for the "no posts available" paragraph. We don't want to show this too early (if we're in the middle of fetching posts).
We also want to output posts if we're currently not fetching. So we'll also add it to the unordered list. 
We can create a paragraph Loading... (or use a nice css spinner) that displays when isFetching is true.
Firebase is really fast, so we'll only see the Loading for a fraction of a second.

# using a service for Http requests
In bigger applications (and maybe in this one), it's a nice practice to outsource data to a service.
Then the components can be mostly lean (and mostly focus on template work).

So we'll create a posts.service.ts file and export the class PostsService{}

We can use @Injectable, and pass an object providedIn: root

In this service we want to have the HTTP request methods and we only want to get the responses in the frontent.
We'll make a new method, createAndStorePost(), and also fetchPosts()

So we'll grab the code from the onCreatePost method in the app component file and move it to the posts service. 

For this to work, we'll need to inject the http service into the posts service.
So we add a constructor and we inject the HttpClient and store it in a property named http. (We also import HttpClient from angular).

We also need to have a property called postData and set it to type Post, which we need to import from the model.
We'll need to set a title property (equal to the title we're getting as an argument) and a content property (equal to the content we're getting as an argument). The right side of the colon refers to the arguments. 
const postData: Post = {title: title, content: content}

Now we can call createAndStorePost to send the request.

We need to inject the posts service into the component:
constructor(private http: HttpClient, private postsService: PostsService) {}

(And also import it from the file path).

Then in onCreatePost we simply call this.postsService.createAndStorePost();
We need to pass in the arguments postData.title and postData.content

Let's also move the logic for fetching into the posts service (and out of the app component ts file).
We'll remove the isFethcing and loadedPosts properties because we don't have them here and don't need them here. Those properties belong in the app component because that's where we need them to change what's displayed in the template. 

We do need to import the map operator from RxJS operators.

We can get rid of the fetchPosts method in the app component. 
And where we used to call this.fetchPosts() we can call this.postsService.fetchPosts();
(We'll do that in onFetchPosts and in ngOnInit)

If we look in the browser we'll see "No posts available"
Because we're running that code in the service, but we lost the connection between the data we fetched in the service and our template. 
We need to fix that.

# services and components working together
One option is to use a Subject in the posts service where we next() our posts when we got them and we subscribe to that Subject in the app component. 

Another way, and simpler way in this example, is to return the result of our get method and the pipe after that.
So, in the festPosts() method we don't subscribe. Instead we only return the prepared observable.
fetchPosts() {
        return this.http
        ...}

Then we can (and have to) subscribe in the app component. (Without a subscribe no request will actually get sent.)

In the ngOnInit with the postsService fetchPosts, we can subscribe. And we get the posts, we can set this.isFetching to false, and we set this.loadedPosts equal to the posts we get back. 
(We'll set this.isFetching to true right before we start sending the request.)
  ngOnInit() {
    this.isFetching = true;
    this.postsService.fetchPosts().subscribe(posts => {
      this.isFetching = false;
      this.loadedPosts = posts;
    });
  }

We'll also copy this code into the logic for onFetchPosts()
We could outsource this into a separate private method to avoid the code repetition if we wanted to.

So now we've moved the result handling into the component, but the more heavy lifting, the part detached from the template and the UI which is sending the request and the transformation of the data, now lives in the service.

This is the best practice when working with Angular and HTTP requests. We move the part that is related to the template (in this case, managing the loading status and managing the loaded data into the component) and be informed about the result of the HTTP request by subscribing in the component, but move the rest into the service and simply return the observable there so that we set up everything in the service, but we can subscribe in the component.

We use a different pattern for creating a post. There, we're subscribing in the service. If our component doesn't care about the response and whether the request is done or not (as is the case here), there is no reason to subscribe in the component. 

# sending a delete request
When the onClearPosts() method runs we want to delete the posts.

We'll set up the request in the postsService.
We'll make a new method called deletePosts() {}
We use the this.http.delete(). The method requires a URL. Since we want to clear all the posts, we'll use the url that targets the overall posts node.

The URL and whether it supports the DELETE method depends on the API we're using. Firebase offers it.
If we want to be informed about that delete in the component, we need to return the observable here in the service and subscribe to it in the component. 
    deletePosts() {
        return this.http.delete('https://academind-http-requests-default-rtdb.firebaseio.com/posts.json')
    }

In onClearPosts(){}:

onClearPosts() {
  this.postsService.deletePosts().subscribe();
}

We want to subscribe in the component so we can also clear the loaded posts array. So we add a method to set the loaded posts to an empty array. 

# handling errors
In Firebase (in the browser) we can go to rules
Right now we're setup in test mode, so read and write access is granted to anyone. We don't need to authenticate any users.
Max changes his read from true to false (the current setup in Firebase is different from what shows in his video). Then trying to fetch data he gets an error.

Also, it stays in the Loading state and a user wouldn't know that the request failed.

We can add an error to teh fetchPosts().subscribe()
As we learned in the Observables section, we can pass more than one argument to subscribe. The second argument is a function that triggers whenever an error is thrown, then we can do something to handle to error to provide a better user experience.

We can add a new property in the app component and set it to null initially.
error = null;
Then we can add a div in the template to say 'An Error Occured' and display that when error is not null.
Below that we can output {{ error }} because that could be our error message.
      <div class="alert alert-danger" *ngIf="error">
        <h1>An Error Occurred!</h1>
        <p>{{ error }}</p>
      </div>

As soon as error gets set to some string, it becomes trueish and the div will show up on the page.

We also want to change the conditions for Loading... to show up if we're fetching and we don't have an error. 
<p *ngIf="isFetching && !error">Loading...</p>

Then we'll set the error in the error handling function (in the subscribe method).
  onFetchPosts() {
    this.isFetching = true;
    this.postsService.fetchPosts().subscribe(posts => {
      this.isFetching = false;
      this.loadedPosts = posts;
    }, error => {
      this.error = error.message;
    });
  }

By default the error object has a message. (Sometimes more helpful than others - we can tweak that and send our own error message.) Firebase does give a different error key with a better message. We can dive into the error object getting returned to see more info about what's happening. There's a status also, etc.

We need to add the second argument in the subscribe in the ngOnInit also.

# Using Subjects for Error Handling
This is a different way to handle errors.
Maybe a use case would be when we send a request and don't subscribe to it in our component.

In our project example when we create a new post we subscribe in the service.
One option would be to return the observable in the service and subscribe in the component (that wouldn't be wrong).

However, we could use a Subject, which is especially useful if we have multiple places in the application that might be interested in the error.

So we could create a new error property in the posts.service.ts file and set it equal to a new Subject (which needs to be imported from rsjs), and that will give us a string (the error message).
error = new Subject<string>();

Then in the createAndStorePost function, in the subscribe, where we get that error, we use the Subject and call next() and pass an error message.
    createAndStorePost(title: string, content: string) {
        const postData: Post = {title: title, content: content}
        this.http
        .post<{name: string}>(
            'https://academind-http-requests-default-rtdb.firebaseio.com/posts.json',
            postData
        )
        .subscribe(responseData => {
            console.log(responseData);
        }, error => {
            this.error.next(error.message);
        });
    }

Then we need to subscribe to that Subject in all the places we're intersted in that error message. In this example we'll subscribe in the ngOnInit:
this.postsService.error.subscribe(errorMessage => {
  this.error = errorMessage;
});

It's a good and recommended practice to unsubscribe if we get rid of this component, so we create a new private property errorSub (in the app component):
private errorSub: Subscription;
We need to import Subscription from rxjs. 
Then we store our subscription in that property (in ngOnInit).
     this.errorSub = this.postsService.error.subscribe(errorMessage => {
      this.error = errorMessage;
    })

Then we import OnDestroy from '@angular/core' (and add it in the implements part of the export class) and in the ngOnDestroy we unsubscribe.
  ngOnDestroy(): void {
    this.errorSub.unsubscribe();
  }

# Using the catchError operator
In the postsService we can import catchError from 'rxjs/operators'

In this example we can add it in the fetchPosts method in the pipe after map.
We add the catchError operator and we get our error response here.
We get the same data we'd get in the second argument of the subscribe method.
Then in there we could send to analytics server, etc, any generic error handling tasks.
We could use the Subject and next() the error message here, too; but maybe we have behind-the-scenes stuff we want to do when an error occurs. Log it, send it to our own server, etc.
Once we're done handling the error we should pass it on. 

So we need to create a new Observable that wraps that error. So we import throwError from rxjs. That is a function that will yield a new observable by wrapping an error. So then we return the observable that is created by throwError and throw the error response.
            catchError(errorRes => {
                // send to analytics server, etc
                return throwError(errorRes);
            })

# Error handling and UX
We can add a button in our alert for the user to say ok and get rid of the error message. We can have a click listener with onHandleError()
<button class="btn btn-danger" (click)="onHandleError()">Okay</button>

Then in the app component TS file we add the method to set this.error back to null.
  onHandleError() {
    this.error = null;
  }

Also, the best thing to do is to reset the isFetching back to false whenever we get an error. So we'll put that in the error function that's part of the onFetchPosts method.
this.isFetching = false;
We'll put it in the ngOnInit also.

# Setting Headers
When sending an HTTP requests, we set the URL, and the data we want to attach. Sometimes we also need to set special headers, like authorization, or content type, or the API needs it.
Any HTTP method (GET, POST, etc) has an extra last argument which is an object where we can configure the request. 
We can configure a lot of things, but we'll start with headers.
Headers takes an object, it's an HttpHeaders object that we need to import from '@angular/common/http'
We create a new instance of this object, and to this object we can pass a JS object with key/value pairs of our headers.
        .get<{[key: string]: Post}>(
            'https://academind-http-requests-default-rtdb.firebaseio.com/posts.json',
            {
                headers: new HttpHeaders({ 'Custom-Header': 'Hello' })
            }
            )

# Adding Query Params
Depending on the API we're working with, we can set query params to the API endpoints. Firebase supports one.
We set parameters by adding the params key in the same config object where we added headers. And we set it to a new HttpParams object (also imported from '@angular/common/http').
On the HttpParams() we call set, and we can set a param name and a value.
We can set it to print (a query param supported by Firebase) and pretty. This just changes the format in which Firebase returns its data.
{
  headers: new HttpHeaders({ 'Custom-Header': 'Hello' }),
  params: new HttpParams().set('print', 'pretty')
}

Then when we run fetchPosts and look in the Network tab we'll see that ?print=pretty got added to the end of the URL.

To attach multiple params, we create a searchParams object and append the next params to it. (Look this up more if needed.)

# observing different types of responses
Sometimes we need the whole response object. Maybe we need the headers or the status, etc. In these cases we can change the way Angular parses the resonse and tell it to give us the whole object instead of the unpacked, extracted response data (from the body).
To do that we add an extra argument to configure the request (in this example we'll do it in our post request).
We change the observe key. observe takes a couple of values. 'body' is the default.
That means we get the response data extracted and converted to a JS object automatically.
We can change this to 'response'
        .post<{name: string}>(
            'https://academind-http-requests-default-rtdb.firebaseio.com/posts.json',
            postData,
            {
                observe: 'response'
            }
        )
With this we'll get back the full HttpResponse object. (in the console.log(responseData))
We can still get access to the body with responseData.body

In the deletePosts function we can use observe again, but this time we'll change it to 'events'
We need to import an operator from rxjs in order to be able to look at events: tap
tap allows us to execute some code without altering the response. We can do somethign with the response but not disturb our subscribe function and the functions we passed as arguments to subscribe.
So we add .pipe(tap) and we get an event.
    deletePosts() {
        return this.http.delete('https://academind-http-requests-default-rtdb.firebaseio.com/posts.json',
        {
            observe: 'events'
        }
        ).pipe(tap(event => {
            console.log(event);
        }))
    }

It won't interupt the normal data flow. We don't need to return anything. It just taps in there to do something but lets the response pass through.

Then if we console.log(event) when we clear the posts, we get two outputs, two events. One is an object with type: 0, and the other is the HttpResponse object (which has a type: 4).

Different types of events are encoded with numbers.
If we need really granular control over how we update the UI we can check the event types with an enum we can import from '@angular/common/http'. This is supported in TS only, and for JS it is just a map of numbers. It understands which number stands for which type of event.
if (event.type === HttpEventType.Sent)
there's also .Response, .User, etc.

We could check whether we get back the response, and if so we can log the event body.
if (event.type === HttpEventType.Response) {
  console.log(event.body);
}

We could use if (event.type === HttpEventType.Sent) and then run some code to update the UI to let the user know the data was sent.

# Changing the response body type
We can also configure the responseType. The default here is 'json', which means the response data. This tells Angular it should parse it and convert it to a JS object.
We can set the responseType to 'text' which lets Angular know it will be text and to not try to convert it to a JS object. It could be a blob if it's a file. (We can learn more in the docs.)
{
  observe: 'events',
  responseType: 'text'
}

# introducing interceptors
Right now the way we've set up our code, whenever we want to configure something like search params, we're doing it on a per request basis. And often this is the setup we want. 
However, there might be cases where we want to configure, for example, the header for all of our outgoing requests. For example, when we need to authenticate a user. We wouldn't want to manually configure every request.

We'll create a new file. Called auth-interceptor.service.ts
(This isn't using real authentication, but it is an example.)

We'll export a class AuthInterceptorService with implements HttpInterceptor (which we need to import from '@angular/common/http')
This interface forces us to use the intercept method, which will get two arguments:
first - request object of type HttpRequest (that we need to import); it's a generic type, and we'll set this to <any>
second - next: a function that will forward the request becayse the interceptor will run code before the request leaves our app, before it is really sent, and right before the response is forwarded to subscribe, so we need to call next to let the request continue on its journey. next is of type HttpHandler (which also needs to be imported)

So this code will run right before the request leaves our application.
For example, we could console.log('Request is on its way');
then we should return the result that next gives us. next is an object with an important method that will allow us to let the request continue its journey - the handle method we have to call to which we pass the request object.
export class AuthInterceptorService implements HttpInterceptor {
    intercept(req: HttpRequest<any>, next: HttpHandler) {
        console.log('Request is on its way');
        return next.handle(req);
    }
}

Then we have to provide that service.
In the app.module.ts file we add a new element to the providers array: a JS object with three keys. 
1. provide, which is of type HTTP_INTERCEPTORS (which needs to be imported from '@angular/common/http')
This is the token by which this injection can later be identified by Angular. It will know that all the classes we provide on that token, using that identifier, should be treated as HTTP interceptors and should run their intercept method whenever a request leaves the application.
2. useClass, where we now point at the interceptor class we want to add 
Here that would be the AuthInterceptorService (which we also need to import)
3. multi, and we set it to true (this lets Angular know this interceptor should not replace other interceptors)
providers: [
  {
    provide: HTTP_INTERCEPTORS,
    useClass: AuthInterceptorService,
    multi: true
  }
],
This is a dependency injection supported by Angular that allows us to register a service under a different identifier.
Angular will do the rest; it will automatically grab all our HTTP interceptors and run their intercept method whenever a request leaves the application.
This code (the console.log(Request on it's way)) runs for every request that leaves the app. If we want to restrict the requests this executes on, we can do that in the interceptor (the service file).
We can do an if with the req.url (since we have the request object here), and only send with requests to a certain url if we want.

# Manipulating request objects
Inside an interceptor we can modify the request object. However, the object itself is immutable. So we have to create a new one and call req.clone. And inside the clone method we can pass in a JS object where we can overwrite the core things. New url, new headers, new params, etc.
We can use req.headers.append() to keep all the original headers and add a new one.
const modifiedRequest = req.clone({headers: req.headers.append()})
we can put Auth as the key and xyz as the value.
const modifiedRequest = req.clone({headers: req.headers.append('Auth', 'xyz')})
Then in the next.handle() we don't forward the original request, but the modifiedRequest.
That's a typical use case for an interceptor. We change the request and then forward the modified request.
(And if we only wanted to append the header to certain requests, we could use an if check with the req.url, for example)

# response interceptors
We not only have access to request interceptors; we can use response interceptors.
We do this by adding something to handle(), because handle actually gives us an observable. (Our request is an observable to which we subscribe)
So in the handle we get teh request with the response in it wrapped in an observable.
So, we can add .pipe() and do something with the response if we want.
We could add the map operator to change the response (but make sure to not change it in a way that the rest of the app breaks).
Or we can use tap (which we need to import) in order to look into the response.
Here in tap we get an event (always).
We can check if the event type is equal to the HttpEventType (that we need to import) of type Response.
Then we can console.log that the response has arrived and console.log the event body (which will be the response body.)
        return next.handle(modifiedRequest).pipe(tap(event => {
            if (event.type === HttpEventType.Response) {
                console.log('Response arrived, body data: ');
                console.log(event.body);
            }
        }));

# multiple interceptors
(we'll see interceptors again in the authentication section)
We can add another interceptor: logging-interceptor.service.ts
export class LoggingInterceptorService implements HttpInterceptor {
    intercept(req: HttpRequest<any>, next: HttpHandler) {
        console.log('Outgoing request');
        console.log(req.url)
        return next.handle(req).pipe(tap(event => {
            if (event.type === HttpEventType.Response) {
                console.log('Incoming response');
                console.log(event.body);
            }
        }));
    }
}

Then we can take out the logging in the auth interceptor.
So the auth interceptor is attaching the auth header and the other interceptor is repsonsible for logging.

The order we provide the interceptors matters because that's the order they'll be exectuted. (Also need to import the LoggingInterceptorService from the file path)
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptorService,
      multi: true
    },
    {
      provide: HTTP_INTERCEPTORS,
      useClass: LoggingInterceptorService,
      multi: true
    }
  ],

# Official docs
https://angular.io/guide/http
