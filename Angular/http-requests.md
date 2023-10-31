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

If we want to store all the posts in an array the we can then loop over, we would need to transform the object to an array. 
For that we'll use an observable operator.