# Services
Another class we can add that acts as a central repository or business unit to centralize our code (in orde to not repeat the same code in mulitple places or to pass data to the components that need it)

# Creating a new service (loggingService)
We can create it in the app folder or a shared subfolder (depending on where the service is going to be used).
We'll name this one logging.service.ts
In the file we'll export a class, and name it LoggingService

A componenet becomes a component because we add the @Component decorator. Same thing for a directive.
For a service, we don't need to add a decorator. It's just a normal TS class. 

So we can create a helper method, logStatusChange() and take the new status as a parameter: logStatusChange(status: string) {}
Then we can do a console.log()

With that we've centralized the code, and now we need to use this service in our app.

We do not use a service by importing it into a compoenet and creating a variable with a new Service() which then calls the method we created. Angular provides a much better way of getting access to the services. We should not create the instances manually.

# Injecting the Service into Components
Angular has a tool called the dependency injector. 
A dependency is something a class of ours will depend on, for example the new account component depends on the Logging Service because we want to call a method in that service, and the dependency injector injects this dependency -- injects an instance of this class -- into our component automatically. 
We just need to inform Angular that we require such an instance.

We add a constructor to the component where we want to use our service. Then we bind it to a property using the TS shortcut private -- adding an accessor in front of the name of the argument to istantly create a property with the same name and bind the value to it. Then we have to add a type, and it has to be the name of the service we want to inject here. And we need to add the import at the top.

In the new account component:
constructor(private loggingService: LoggingService) {}

This informs Angular that we will need an instance of this logging service.

When Angular comes across our selectors in the code, it gives us instances of our components. Since Angular is responsible for instantiating our components, Angular will need to contruct them correctly. So if we define in the constructor that we require some argument Angular will recognize this and try to give us that argument. In this case, it knows we want an instance of the logging service class becuase we defined the type here. 

Now it knows what we want, but not yet how to give it to us. We need to provide a service. Provide simply means we tell Angular how to create it. We add one extra property to the @Component({}) decorator.
providers: []
This takes an array, and here we have to specify the type of what we want to be able to get provided. 
providers: [LoggingService]

With that, when Angular analyzes the component, it will recognize that it should be able to give us a logging service and it will set itself up to be able to do that. 

Then in the component we can access our logging service property and call logStatuschange()
this.loggingService.logStatusChange(accountStatus)

We're not creating the instance manually, Angular does it for us. This is better because it lets us stay in the Angular ecosystem and Angular knows how our app works. (Other advantages will become apparent later.)

Overall, using this service makes our code a bit leaner, and will matter a lot in bigger applications. 

# create a new service (Data)
Typical use case for a service - to store and manage data

Instead of storing data in the app component and then having a chain of property and event binding to get data to the app component so we can update our accounts - let's create a service for that. 

We can take the array of accounts from the app component and put it into the accounts.service.ts file. Then we add an addAccount() method where we expect to get an account name and status, and an updateStatus method() where we get the id of the account we want to update and the new status. 
The logic for these two methods is basically the same as what we had in the app component. For the addAccount we want to push the new account onto the accounts array and pass an object with the name and status. 

Then we don't need those methods in the app component. However, we do need to set the accounts variable, and set it to an array of objects, with the type {name: string, status: string}. And we start with that set to an empty array.

Then we need to inject the service with the constructor.
constructor(private accountsService: AccountsService) {} (and that will need to be imported from the acocunts.service file).
We also need to add a provider.

Then we add the OnInit hook since most initializations should be done not in the constructor but in ngOnInt(). We set this.accounts to this.accountsService.accounts

Since accounts is an array, it is a reference type. And by setting it equal we're actually getting access to the exact same array as stored in the service. 

* All of the above will not actually work because it's not using services correctly. *

# Understanding the Hierarchical Injector
The Angular injector is hierarchical - Angular knows to provide a service for a component and all its child components. They will all receive the same istance of this service. 
The highest level to provide a service is in the AppModule - the same instance of the service is available application-wide.
The next level is the AppComponent. The same instance of the service is available for all components, but not for other Services. (Side note: we can inject services into services.)
The lowest level is a single component with no child components. The component will have its own instance of the service. This will actually override a service provided at a higher level. 

# How many instances of service should it be?
We might have cases where we want to have many different instances of the same service, where we don't want to have the same instance. 
In this case we want to have the same instance. 
The way we wrote it (above) we have three instances. The instance in app component gets provided to both the account and new account components, so we don't need to provide it again and override the first instance (the one we would get from the app component).

We can fix it by removing the service from the providers array in the account component and the new-account component. We don't remove it from the constructor, because that tells Angular that we want the instance of the service here.
