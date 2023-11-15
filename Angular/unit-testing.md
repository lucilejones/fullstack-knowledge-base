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

# Adding a component and some fitting tests
In the User component test file:

import { TestBed } from '@angular/core/testing';
import { UserComponent } from './user.component';

describe('Component: User', () => {
    beforeEach(() => {
        TestBed.configureTestingModule({
            declarations: [UserComponent]
        });
    });

    it('should create the app', () => {
        let fixture = TestBed.createComponent(UserComponent);
        let app = fixture.debugElement.componentInstance;
        expect(app).toBeTruthy();
    });
});

# Testing dependencies: components and services
Max creates a simple user service. To add that to the test, we import the UserService in to the user testing folder and add the following:
it('should use the username from the service', () => {
    let fixture = TestBed.createComponent(UserComponent);
    let app = fixture.debugElement.componentInstance;
    let userService = fixture.debugElement.injector.get(UserService);
    expect(userService.user.name).toEqual(app.user.name);
});

But one thing is missing here that happens automatically in the browser. We need to add change detection (to update our properties, etc after the injection).
fixture.detectChanges();

it('should use the username from the service', () => {
    let fixture = TestBed.createComponent(UserComponent);
    let app = fixture.debugElement.componentInstance;
    let userService = fixture.debugElement.injector.get(UserService);
    fixture.detectChanges();
    expect(userService.user.name).toEqual(app.user.name);
});

Then testing for if the username displays when the user is logged in but doesn't display when the user isn't logged in.
We don't need the injector for this test, but we do need to set isLoggedIn to true.

it('should display the username if user is logged in', () => {
    let fixture = TestBed.createComponent(UserComponent);
    let app = fixture.debugElement.componentInstance;
    app.isLoggedIn = true;
    fixture.detectChanges();
    let compiled = fixture.debugElement.nativeElement;
    expect(compiled.querySelector('p').textContent).toContain(app.user.name);
});

it('shouldn\'t display the username if user is not logged in', () => {
    let fixture = TestBed.createComponent(UserComponent);
    let app = fixture.debugElement.componentInstance;
    fixture.detectChanges();
    let compiled = fixture.debugElement.nativeElement;
    expect(compiled.querySelector('p').textContent).not.toContain(app.user.name);
});

# Simulating async tasks
Max creates a DataService to simulate an async operation
We don't want to call the actual getDetails method because we don't want to actually reach out to a server during testing. We'll create fake implementation.
spyOn means we listen to a method that we specify.
We would expect data to be undefined in the beginning, but then the data should change during run time. We have to use the async function (that's imported from angular/core/testing) and wrap the callback with that function.
Then in the test file:

it('should fetch data successfully if called asynchronously', async(() => {
    let fixture = TestBed.createComponent(UserComponent);
    let app = fixture.debugElement.componentInstance;
    let dataService = fixture.debugElement.injector.get(DataService);
    let spy = spyOn(dataService, 'getDetails')
        .and.returnValue(Promise.resolve('Data'));
    fixture.detectChanges();
    fixture.whenStable().then(() => {
        expect(app.data).toBe('Data');
    });
}));

# using "fakeAsync" and "tick"
Since these tests are only theoretically asynchronous, we can use another method.
We can use fakeAsync as a wrapper. (Which also needs to be imported from the testing package)
Then we don't need the whenStable funciton. But we call tick() - which also needs to be imported - in between detect changes and the expect. tick means to finish all asynchronous tasks now.
it('should fetch data successfully if called asynchronously', fakeAsync(() => {
    let fixture = TestBed.createComponent(UserComponent);
    let app = fixture.debugElement.componentInstance;
    let dataService = fixture.debugElement.injector.get(DataService);
    let spy = spyOn(dataService, 'getDetails')
        .and.returnValue(Promise.resolve('Data'));
    fixture.detectChanges();
    tick();
    expect(app.data).toBe('Data');
}));

# isolated vs non-insolated tests
For some functionality, like a pipe that just reverses a string, or a service that just transforms some data, we don't need the Angular 2 testing package. Because we can test those kinds of things in isolation, we can write just a normal unit test.

An isolated test for the reverse pipe:
import { ReversePipe } from './reverse.pipe';

describe('Component: User', () => {
    it('should reverse the string', () => {
        let reversePipe = new ReversePipe();
        expect(reversePipe.transform('hello')).toEqual('olleh');
    });
});

# resources
Docs: https://angular.io/docs/ts/latest/guide/testing.html
