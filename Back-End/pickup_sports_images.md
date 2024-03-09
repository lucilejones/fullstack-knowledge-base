# Create Event Details View
When a user clicks on "View More"

We'll send a request to localhost:3000/events/:id to trigger the show action.

Then we'll create a new view in the event_blueprint.rb:
    view :long do
        fields :title, :start_date_time, :end_date_time, :guests, :sports, :content
        association :user, blueprint: UserBlueprint, view: :normal
    end

We'll add more, but for now we'll just add the content.

Then in the events_controller we'll change what renders in the show action (using the EventBlueprint):
    def show
        render json: EventBlueprint.render_as_hash(@event, view: :long), status: :ok
    end

Then in the frontend we'll generate a new component:
ng g c features/event-details

Then in app.routes.ts we'll create a route for this component:
[the path doesn't have to match the path in the backend]
    {
        path: 'events/:id',
        loadComponent: () => import('./features/event-details/event-details.component').then((c) => c.EventDetailsComponent),
        canActivate: [authGuard]
    },

In the event.component.html file:
<button class="view-more-button" [routerLink]="['/events', event.id]">View More</button>

We need to add RouterLink to the TS file (in the imports)

Then if we click on "View More" we should see event-detail works!

Next we'll add to our event.service.ts file:
getEvent(id:string | number) {
    return this.http.get<Event>(`${environment.apiUrl}/events/${id}`)
}

Then in the event-details.component.ts file:
event:Event = new Event({});

constructor(private route:ActicatedRoute, private eventService:EventService) {}

ngOnInit(): void {
    this.route.params.subscribe((params) => {
        this.eventService.getEvent(params['id']).subscribe({
            next: (event:Event) => {
                this.event = event
            },
            error: (error) => {
                console.log(error);
            }
        })
    })
}
[we need to make sure to import the Event model]

Then we'll create the HTML for the event-details.component:
<div class="event-details-container">
    <h2 class="event-title">{{ event.title }}</h2>
    <p class="event-content"> {{ event.content }}</p>
    <div class="event-info">
        <span class="event-dates">
            {{ event.start_date_time | date: "medium" }} - {{ event.end_date_time | date: "medium" }}
        </span>
        <div class="event-creator">
            created by: <strong>{{ event.user.username }}</strong>
        </div>
    </div>
    <div class="sports-container">
        <h3>Sports:</h3>
        <ul>
            @for(sport of event.sports; track sport.id) {
                 <li>{{ sport.name }}</li>
            }
        </ul>
    </div>
</div>

We'll also need to import the date pipe into the TS file.

We need to create a Sport model (to then be able to include in the event model).

sport.ts:
import { User } from "./user";

export class Sport {
    id: number;
    name: string;

    constructor(event:any) {
        this.id = event.id || 0;
        this.name = event.name;
    }
}

Then in the Event model we add:
sports: Sport[]
...
this.sports = events.sports;

[Here German ran into an error where the program thought there was already a server running. He had to look for which one that was (that had crashed) and run a command to kill it, then start a new one]

Then we'll add styling to the SCSS file:
.event-details-container {
    max-width: 800px;
    margin: 2rem auto;
    padding: 20px;
    background-color: #f8f8f8;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.event-title {
    color: #333;
    background-color: #AFD8E6;
    padding: 10px;
    border-radius: 4px;
}

.event-content {
    margin: 20px 0;
    line-height: 1.6;
}

.event-info {
    display: flex;
    justify-content: space-between;
    background-color: #ffa500;
}

.event-info .event-dates, .event-info, .event-creator {
    margin: 0px;
}

.event-timestamps {
    font-size: 0.86rem;
    color: #666;
    margin-top: 10px;
}

.event-timestamps span {
    display: block;
    margin: 5px 0;
}


# Uploading images locally with Postman
We'll use Active Storage

rails active_storage:install

This will set it up and create a migration file.
Then we run rails db:migrate

Then to the event.rb we add:
has_one_attached :cover_image

In the events_controller.rb we need to include the :cover_image in the event_params.

Then in postman we can send a POST with the file as form-data.
[We can also include the other key/values in the form data, ex. title, content, start_date_time, etc]

Then in the rails console we can test whether it's attached:
Event.last.cover_image.attached?

[I keep getting false when it should be true...]


# Get image Url
First we define a method in the event model.
app/models/event.rb:
def cover_image_url
    rails_blob_url(self.cover_image, only_path: false) if self.cover_image.attached?
end

We want to use the url helper, so in the event.rb file we need to:
include Rails.application.routes.url_helpers

In order to get the correct full path, we need to specifiy the :host

In config/environments/development.rb:
Rails.application.routes.default_url_options[:host] = "localhost:3000"
[We'll need to make sure to restart the server]

Then we'll update the EventBlueprint:
view :long do
... , :cover_image_url
...

In the response back (if we do a GET request to events/:id) we'll get the url starting with localhost. We can copy and paste it into the browser and we'll be able to see the image. 


# Setting up our Create Event Component
First we'll create the component:
ng g c features/crete-event

In the app.routes.ts file:
    {
        path: 'create-event',
        loadComponent: () => import('./features/create-event/create-event.component').then((c) => c.CreateEventComponent),
        canActivate: [authGuard]
    },

Then in the navigation component html, in the sidebar, we'll add:
<a routerLink="/create-event">Create Event</a>

In the create-event html we'll set up the form:
eventForm: FormGroup = new FormGroup({
    title: new FormControl(""),
    content: new FormControl(""),
    start_date_time: new FormControl(""),
    end_date_time: new FormControl(""),
    guests: new FormControl(""),
    sportIds: new FormArray([])
})

We'll need to import FormArrary, FormControl, and FromGroup from '@angular/forms'

We'll implement OnInit. Then:
  ngOnInit(): void {
    this.loadSportIds();
  }

We'll create that method and also import the Sport model.

sports: Sport[] = []

loadSportIds() {
    #add a get request to get the sports ids
}

In the backend, we'll need to set up a route.
In the config/routes/rb file:
resources :sports

Then we make a sports_controller.rb file:
class SportsController < ApplicationController
    before_action :authenticate_request
    
    def index
      sports = Sport.all
      render json: SportBlueprint.render(sports), status: :ok
    end
  end

And a simple blueprint:
class SportBlueprint < Blueprinter::Base
    identifier :id
    fields :name
end

Then we'll create a service:
ng g s core/services/sport

In the SportService:
getSports() {
    return this.http.get(`${environment.apiUrl}/sports`)
}

We'll need to inject the HTTPClient and the environment.

Then in the create-event TS we'll inject the SportService and define the loadSportIds method:
this.sportService.getSports().subscribe({
    next: (res) => {
        console.log(res);
    },
    error: (error) => {
        console.log(error);
    }
})

Then (in the create-event component) we'll create a method to add a Sport:
addSportToForm() {
    (this.eventForm.get("sportIds") as FormArray).push(new FormControl(false))
}

This will iterate through the array of sports ids and create an entry for each, but we want to start out as false, so it's not automatically checked. 


In the loadSportIds method, instead of just console logging:
next: (sports:any) => {
    this.sports = sports;
    sports.forEach((sport:Sport) => {
        this.addSportToForm()
    })
}

Then we'll define a method to get the FormControls:
get sportIds(): FormArray {
    return this.eventForm.get("sportIds") as FormArray
}

We can move the styles from the auth.shared.scss file into the main styles.scss file.

We need to import the ReactiveFormsModule.

Then in the create-event HTML:
<div class="form-container">
    <form [formGroup]="eventForm" (ngSubmit)="onCreateEvent()">
    <div class="form-group">
        <label for="title">Title</label>
        <input type="text" formControlName="title" />
    </div>
    <div class="form-group">
        <label for="content">Content</label>
        <input type="text" formControlName="content" />
    </div>
    <div class="form-group">
        <label for="guests">Guests</label>
        <input type="number" formControlName="guests" />
    </div>
    <div class="form-group">
        <label for="start_date_time">Start Date and Time:</label>
        <input type="datetime-local" formControlName="start_date_time">
    </div>   
    <div class="form-group">
        <label for="end_date_time">End Date and Time:</label>
        <input type="datetime-local" formControlName="end_date_time">
    </div>
    @if(sportIds.length > 0) {
        <div formArrayName="sportIds">
            @for(sport of sports; track sport.id) {
            <input type="checkbox" [formControlName]="$index" [value]="sport.id">
            {{ sport.name }}
        }
        </div>
    }
    <button type="submit">Create Event</button>
    </form>


</div>

onCreateEvent() {
    console.log(this.eventForm)
}

When we console log this on submit and look at the value, we'll see the the sportIds array is [false, true]. When we submit we'll actually want to extract the id of the sports that just have the value of true. 

In the method:
onCreateEvent() {
    const sportIdsFormValue = this.eventForm.value.sportIds;
    const sportIds = sportIdsFormValue.map((checked:boolean, i:number) => {
        return checked ? this.sports[i].id : null
    }).filter((id:any) => {
        return id !== null
    })
    
    const event:Event = {
        sport_ids: sportIds,
        ...this.eventForm.value
    }
    this.eventService.createEvent(event).subscribe({
        next: () => {
            this.router.navigate(['/events']);
        },
        error: (error) => {
            console.log(error);
        }
    })
}

In the event.service.ts we'll add:
createEvent(event:Event) {
    return this.http.post(`${environment.apiUrl}/events`, event)
}


# Uploading images from our front end
In the HTML we'll need to add a label for the cover image and a file input type.
<div>
    <label for="cover_image">Cover Image</label>
    <input type="file" (change)="onFileSelected($event)" id="cover-image">
</div>

Then we define the method onFileSelected() - in the TS file:
onFileSelected(event:any) {
    if(event.target.files && event.target.files[0]) {
        this.selectedFile = event.target.files[0]
    }
}

We'll have a defined property:
selectedFile: File | null = null;

Then we have to update our onCreateEvent and how we're sending the data to the backend. We aren't going to use JSON anymore. We're going to use Form Data.

We'll create a separate method, extractSportIds() and put the logic for the sportIds in there:
extractSportIds() {
        const sportIdsFormValue = this.eventForm.value.sportIds;
    const sportIds = sportIdsFormValue.map((checked:boolean, i:number) => {
        return checked ? this.sports[i].id : null
    }).filter((id:any) => {
        return id !== null
    })
    return sportIds;
}

Then in the onCreateEvent() {
    const sportIds = this.extractSportIds();
    const formData:any = new FormData();
    formData.append('title', this.eventForm.get('title')!.value)
    formData.append('content', this.eventForm.get('content')!.value)
    formData.append('guests', this.eventForm.get('guests')!.value)
    formData.append('start_date_time', this.eventForm.get('start_date_time')!.value)
    formData.append('end_date_time', this.eventForm.get('end_date_time')!.value)
    sportIds.forEach((sportId:any)=> {
        formData.append('sport_ids[]', sportId)
    })
    formData.append('cover_image', this.selectedFile, this.selectedFile!.name)
}

Then, instead of passing in the event, we'll pass in the formData.
this.eventService.createEvent(formData)...

Then we want to be able to include the cover image in the event-details page:
<div class="event-image">
    <img [src]="event.cover_image_url || ''" alt="">
</div>

We can have a default image instead of an empty string.

We'll need to add the cover_image_url to the Event model:
cover_image_url: string;
...
this.cover_image_url = event.cover_image_url;

Then we'll make sure the image only has the container's width:
.event-image {
    width: 100%;
}


# Uploading images via production using Cloudinary
In the backend (API) we add the Cloudinary gem in the production group

group :production do
    gem 'pg'
    gem 'cloudinary'
end

run bundle install

In the config/environments/production.rb file:
config.active_storage.service = :cloudinary

Then in the config/storage.yml file:
cloudinary: 
    service: Cloudinary
    cloud_name: "test"
    api_key: "234804571"
    api_secret: "sdlfkjrt"

[This info comes from Cloudinary - from the account dashboard]

In order to protect this information we're going to edit the credentials file. We can only do that if we have the master.key
run EDITOR="code --wait" bin/rails credentials:edit

[might not be able to use this command - there are notes in the reading material about that other option (usually for windows)]

When the creditials file opens, we paste in the service, plus the cloud_name, api_key, and api_secret.
Then we close the file and we use Rails.application.credentials for the sensitive info:

cloudinary:
    service: Cloudinary
    cloud_name: <%= Rails.application.credentials.cloudinary[:cloud_name] %>
    api_key: <%= Rails.application.credentials.cloudinary[:api_key] %>
    api_secret: <%= Rails.application.credentials.cloudinary[:api_secret] %>

Then in the production.rb file we need to specify the host:
Rails.application.routes.default_url_options[:host] = "https://sitename.com/"