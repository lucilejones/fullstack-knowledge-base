# Allowing users to join or leave events
First we'll update the association in the event.rb to be more specific, saying it belongs to a creator (which points to the user who created the event)

belongs_to :creator, class_name: "User", foreign_key: "user_id"

We'll also rename the users through event_participants to participants

has_many :participants, through: :event_participants, source: :user

Next, we want to set up the routes for users to join or leave events.

In the routes.rb file:
resources :events do
    #localhost:3000/events/1/join
    post 'join', to: 'events#join'
    delete 'leave', to: events#leave'
end

Then we'll define the actions in the events_controller.rb:
def join
    event = Event.find(params[:event_id])

    #check if the current user is the event creator
    return render json: {error: "You can't join your own event."}, status: :unprocessable_entity if event.creator.id == @current_user.id

    #check if the event is full
    return render json: {error: "Event is full."}, status: :unprocessable_entity if event.participants.count >= event.guests

    #check if the current user is already a participant
    return render json: {error: "You are already a participant."}, status: :unprocessable_entity if event.participants.include?(@current_user)

    event.participants << @current_user

    head :ok
end

def leave
    event = Event.find(params[:event_id])

    event.participants.delete(@current_user)
    head :ok
end


# Integrating Pusher to our Ruby on Rails API

We'll add the gem to our Gemfile:
gem 'pusher'

run bundle install

Then we need to access the credentials file and enter the info we get from the Pusher dashboard (in App keys).
EDITOR="code --wait" bin/rails credentials:edit

pusher:
    app_id: "12343"
    key: "sdkj3489"
    secret: "asdfukh345"
    cluster: "us2"

Then in the config/initializers folder, we create a file pusher.rb:
require 'pusher'

Pusher.app_id = Rails.application.credentials.pusher[:app_id]
Pusher.key = Rails.application.credentials.pusher[:key]
Pusher.secret = Rails.application.credentials.pusher[:secret]
Pusher.cluster = Rails.application.credentials.pusher[:cluster]
Pusher.logger = Rails.logger

In the events_controller.rb file:
(in the join method)
Pusher.trigger(event.creator.id, 'notifications', {
    event_id: event.id,
    notification: "#{@current_user.username} has joined #{event.title}!"
})

head :ok
...
(in the leave method)
Pusher.trigger(event.creator.id, 'notifications', {
    event_id: event.id,
    notification: "#{@current_user.username} has left #{event.title}!"
})

head :ok

We'll also want to update the blueprint to events. Change :user to :creator
And we want to add the participants in the long view (and whether the current user has joined).

association :participants, blueprint: UserBlueprint, view: :normal
association :creator, blueprint: UserBlueprint, view: :normal
field :has_joined do |event, options|
    event.has_joined?(options[:current_user])
end

(We'll grab this from the controllers action and define this in our model.)

In the event.rb model:
def has_joined?(user)
    self.participants.include?(user);
end

Then in the events.controller.rb file:
(We pass in the current_user in the show action)
def show
    render json: EventBlueprint.render_as_hash(@event, view: :long, current_user: @current_user), status: :ok
end


# Front End - Allowing people to join or leave an event
We need to update the UI for events on the frontend (since we changed the blueprint in the API). We'll update the event model.

In the event.ts file:
guests: number;
has_joined: boolean;
participants: User[];
creator: User;
...
this.creator = event.creator || new User({});
this.guests = event.guests || 0;
this.has_joined = event.has_joined || false;
this.participants = event.participants || [];

Then we'll need to update the event.component.html:
{{event.creator.username}}

Also in the event-details.component.html:
{{event.creator.username}}

Then we need to add the join/leave functionality in the event.service.ts:
joinEvent(eventId:number) {
    return this.http.post(`${environment.apiUrl}/events/${eventId}/join`, {})
}

leaveEvent(eventId:number) {
    return this.http.delete(`${environment.apiUrl}/events/${eventId}/leave`)
}

Next we'll add a button for a user to be able to join.
But first, we'll want to check if the current user is the event creator.
We'll inject the UserService into the event-details.component.ts:
currentUser: User | null = new User({});
...
[in the ngOnInit]
this.userService.currentUserSubject.subscribe(()=>{
    this.currentUser = this.userService.currentUserSubject.value;
})
[here, German just put currentUserSubject, but my code has currentUserBehaviorSubject]

In the event-details.component.html:
@if(event.creator.id !== currentUser?.id){
    <button class="join-button">Join</button>
}

We'll add a property to the TS file:
hasJoined: boolean = false;

[in the ngOnInit, after the getEvent method]
this.hasJoined = event.has_joined;

Then in the HTML:
<button class="join-button" (click)="toggleJoinEvent()">{{hasJoined ? 'Leave' : 'Join'}}</button>

In the TS file:
toggleJoinEvent() {
    <!-- setup observable of join or leave -->
    const eventJoin$ = this.hasJoined ? this.eventService.leaveEvent(this.event.id) : this.eventService.joinEvent(this.event.id);

    eventJoin$.subscribe({
        next: ()=> {
            this.hasJoined = !this.hasJoined;
        },
        error: (error)=> {
            console.log(error);
        }
    })
}


# Preview Guests for an Event
In the event-details.component.ts ngOnInit:
this.prepareGuests();

...
prepareGuests() {
    this.guests = [...this.event.participants]

    const emptySlots = this.event.guests - this.event.particpants.length;

    for(let i = 0; i < emptySlots; i++) {
        this.guests.push({ empty: true });
    }
}

trackById(index: number, item:any) {
    return item.id || index;
}

Before the constructor we'll define a property:
guests: any = [];

Then we update the toggleJoinEvent() method:
...
this.hasJoined = !this.hasJoined;

if(this.currentUser) {
    if(this.hasJoined){
        this.event.participants.push(this.currentUser)
    } else {
        this.event.participants = this.event.participants.filter((p) => p.id !== this.currentUser?.id)
    }
}

In the HTML:
<div class="guests-container">
    @for(guest of guests; track trackById) {
        <div class="guest-item">
            @if(guest.empty) {
                <div>Open Slot</div>
            } @else {
                <div>{{ guest.username}}</div>
            }
        </div>
    }
</div>

We'll need to execute the prepareGuests function again in order to update the UI.

In the joggleJoinEvent, eventJoin$.subscribe:
this.prepareGuests();

In the event-details.component.scss file:
.guests-contatiner {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    padding: 10px;
}

.guest-item {
    border: 1px solid #ccc;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 200px;
    height: 100px;
}


# Integrating Pusher into our Frontend
We're going to subscribe to pusher channels.

The first thing we'll do is install (in the client:)
npm install pusher-js

Then in the environments/environment.development.ts file:
pusher: {
    key: '',
    cluster: '',
}

We'll also copy that to the production file.

We're going to allow a user to be subscribed to their own channel.

Then we need to create a notification.service.ts file in the services folder.
pusher: any;
channel: any;
...
listen(userId:number) {
    this.pusher = new Pusher(environment.pusher.key, {
        cluster: environment.pusher.cluster
    })

    this.channel = this.pusher.subscribe(userId.toString())

    this.channel.bind('notification', (date:any) => {
        console.log(data);
    })
}
[we'll need to import Pusher from 'pusher-js';]
[In the events.contoller.rb file in the API we have the two Pusher.trigger actions/methods - German updated these to say 'notification' instead of 'notifications'; we're binding to the specific event]

It makes sense to listen to the channel when our app loads.
We can do that in our app.config.ts file:
[In the initializeUserData, in the if authService.isLoggedIn in the subscribe]
return () => userService.getBootstrapData().subscribe((res:any) => {
    const currentUser = res.current_user
    notificationService.listen(currentUser.id)
})

[we'll need to include the notificationService: NotificationService and provide it in the deps array.]

When the user logs in we want to also subscribe/listen. We'll want to make sure to stop when the user logs out. (We'll figure that out later.)

At this point, we'll have to refresh the page in order to start listening. (We haven't set that up to start listening on login, yet.)


# Fixing Login for Pusher
We want the app to subscribe to the pusher channel on login.

In the login.component.ts file:
[in the login function, in the this.authService.login]
next: (res:any) => {
    // subscribe to pusher channel
    this.notificationService.listen(res.current_user.id);
    this.router.navigate(['/]);
}

We need to inject the notification service. 

We want to stop listening when a user logs out.

We need to inject the NotificationService into the authentication.service.ts file.
Then in the logout function:
const currentUser = this.userService.currentUserSubject.value;
// unsubscribe from pusher channel
this.notificationService.unsubscribeChannel(currentUser!.id);

We have to first define a method in the notification.service.ts:
unsubscribeChannel(userId:number) {
    this.pusher.unsubscribe(userId.toString())
}


# Popup Notification
We'll need to dynamically load a component. We can create a component at runtime, then it will go away and get removed from the DOM.

First we'll create the component:
ng g c shared/popup

[we'll want to move it to the components folder]

We won't need the HTML, SCSS, or SPEC files.

In the popup.component.ts file:

We'll change templateUrl to template: and we'll remove the link and just set up a small template.
template: `
    <span>{{ message }}</span>
`

Then in the export class we set a message property:
message: string = ''

Instead of the styleUrl, we'll change to 
styles: [`
    .notification {
        display: block;
        position: fixed;
        top: 20%;
        right: 0;
        z-index: 1000;
        width: 250px;
    }
`]

We'll also add some animations:
animations: [
    trigger("state", [
        state("void", style({
            transform: "translateX(100%)",
            opacity: 0
        })),
        state("opened", style({
            transform: "translateX(0)",
            opacity: 1;
        })),
        transition("void => "opened", animate("300ms ease-out")),
        transition("opened => void", animate("300ms ease-in"))
    ])
],

[we'll need to import trigger from '@angular/animations'; also state and style, and transition and animate]

We'll give the span a class="notification"

We actually get rid of the property messages and instead:
@HostBinding('@state')
state: "opened" | "void" = "void";

@Input()
set message(msg:string) {
    this._message = msg;
    this.state = "opened"
}

get message(): string {
    return this._message
}

private _message = "";

@Output()
closed = new EventEmitter<void>();

Then we'll create a service:
ng g s core/services/popup

In the popup.service.ts file:
showAsElement(message:string) {
    const popupElement: NgElement & WithProperties<PopupComponent> = document.createElement("popup-element") as any;

    // setTimeput to remove element

    popupElement.message = message;
    document.body.appendChild(popupElement);
}

However, we don't have access to these types. 
We need to install a specific package - 
npm install @angular/elements@17.1.2 --save
[German was able to install 17.1.2; I had to do 17.1.3]

Then we'll import NgElement and WithProperties from '@angular/elements'

In the app.component.ts file:
constructor(injector:Injector, public popup:PopupService) {
    const popupElement = createCustomElement(PopupComponent, {injector})
    customElements.define("popup-element", popupElement)
}

We'll need to import Injector, PopupService, createCustomElement, and PopupComponent.

This new way is easier than the old way of creating elements at runtime.

Then in the notification service, we inject the popupService.
Then we'll trigger the popup in the this.channel.bind (when we get the notification from Pusher.)

this.popupService.showAsElement(data.notification)

Then we'll include the setTimeout in the popup.service.ts file:
setTimeout(() => {
    document.body.removeChild(popupElement);
}, 3000)

We are getting an error related to the BrowserModule.
We need to add (in the app.config.ts file, in the providers array):
provideAnimations(),
[and we need to import it at the top from '@angular/platform-browser/animations]
