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
