# what are pipes?
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
