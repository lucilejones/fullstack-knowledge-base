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

    }
</div>