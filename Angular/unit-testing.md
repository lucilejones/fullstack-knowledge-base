# Why unit tests?
Questions we might want to ask:
Does the component work as intented? Does the pipe or service work as intended? Does the Input() or the Injection work as intended?
Of course, and important part will be writing tests correctly.
(And that is the subject for a whole other course. This is just an introduction for how to use testing in Angular.)

We can guard against breaking changes, analyze code behavior for both expected and unexpected results, and we can reveal design mistakes.

Official Docs: https://angular.io/docs/ts/latest/guide/testing.html


# Analyzing the testing setup (as created by the CLI)
In the app.component.spec.ts file.
Each block that starts with it is a test.
the beforeEach() executes some code which should be run before running each test

We want to run the same thing that will run in the browser in the testing environment, which is just a script running. In order to simulate the same behavior, we also need to bootstrap our application. We need to set up the app module and then execute certain tasks the user might do or see.

In the spec file we're importing from @angular/core/testing - the testing package Angular ships with which has some utility toos, for example, TestBed and async.

Then we describe the to-be-tested unit - here it's the app module and the app component.
We then have a closure - the function executed as the second argumnet in describe() - and that will all be executed by the test runner we're using. We'll use the CLI with a command to run these tests. (A typical test runner is Karma - a package we can install)

Then we execute beforeEach()
And each block is executed totally independent of the block before it.
Then we configure a testing module and declare which components we want to have in the testing environment.

Then we have a few tests. The first one checks is the app was properly created. 
We need to use createComponent() for each test since they're independent. And we store the created component in the fixture variable.
Then we get our app by using this fixture that holds our created component. And we use debugElement.
We always end the block using the expect() method. It's related to the testing package
So we pass whatever we're testing, here we pass app to expect, and say we expect it .toBeTruthy()
(Max then goes over the other two basic tests)

# Running Tests (with the CLI)
ng test

