# Frontend setup
updated to version 17.1.2
ng new pickup_sports_client
Stylesheet: SCSS
[we can use plain css in scss files]
Server-side rendering and static site generation aren't necessary

Generate our models:
ng g class shared/models/user

export class User {
    id: number;
    first_name: string;
    last_name: string;
    email: string;
    username: string;

    constructor(user:any) {
        this.id = user.id || 0;
        this.first_name = user.first_name || "";
        this.last_name = user.last_name || "";
        this.email = user.email || "";
        this.username = user.username || "";
    }
}

ng g class shared/models/post

import { User } from './user'; 

export class Post {
    id: number;
    title: string;
    content: string;
    created_at: string;
    user?: User;

    constructor(post:any) {
        this.id = post.id || 0;
        this.title = post.title || "";
        this.content = post.content || "";
        this.created_at = post.created_at || "";
        this.user = post.user || null;
    }
}


ng g c features/timeline --standalone
[standalone - to have a component that's not dependent on a module]

In the timeline.component.ts file:
...imports and @Component({})...
import { Post } from 

export class TimelineComponent {
    posts: Post[] = [
        new Post({
            id: 1,
            title: "Post 1",
            content: "Content 1",
            created_at: "2021-01-01",
            user: {
                first_name: "Jane",
                last_name: "Doe",
                email: "email@email.com",
                username: "jandoe123"
            }
        })
    ]

    constructor(){}
}

In the timeline.component.html file:
<div class="container">
    <div class="side-column"></div>
    <div class="main-column"></div>
    <div class="side-column"></div>
</div>

In the timeline.component.scss file:
.container {
    display: flex;
    justify-content: space-between;
    width: 100%
}

.side-column {
    width: 20%;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.main-column {
    width: 50%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

In the app.routes.ts file:
import { Routes } from '@angular/router';

export const routes: Routes = [
    {
        path: "",
        pathMatch: "full",
        loadComponent: () => import('./features/timeline/timeline.component').then((c) => c.TimelineComponent)
    }
];

The app.component.ts file will by default have RouterOutlet in the imports array.

Then we'll need to add it to the app.component.html file:
<router-outlet/>

Then we can run the app:
ng serve --open

Next we'll create a post component:
ng g c shared/components/posts/post --standalone

In the post.component.ts file:

...imports and @Component({})...
We need to add Input from @angular/core
import { Post } from '../../../models/post';

export class PostComponent {
    @Input({required:true}) post:Post = new Post({})
}

Then in the post.component.html file:
<div class="post-header">
    <div class="post-header-left">
        <img src="" alt="">
        <div class="post-header-info">
            <div class="post-header-info-name">{{post.user?.username}}</div>
            <div class="post-header-info-date">{{post.created_at | date}}</div>
        </div>
    </div>
</div>

<div class="post-content">{{post.content}}</div>

In order to use the date pipe we need to add DatePipe to the imports array in the post.component.ts file.

Then in the post.component.scss file:

.post-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px;
    background-color: #f9f9f9;
    border-bottom: 1pm solid #e1e1e1;
}

.post-header-left img {
    width: 50px;
    border-radius: 50%;
    object-fit: cover;
    margin-right: 10px;
}

.post-header-info-name {
    font-size: 1rem;
    font-weight: bold;
    color: #333;
}

.post-content {
    padding: 15px;
    font-family: 'Open Sans';
    line-height: 1.6;
    color: #333;
}

Then we need to update the timeline.component.html file:
<div class="container">
    <div class="side-column"></div>
    <div class="main-column">
        @for (post of posts; track post.id){
            <app-post [post]="post"/>
        }
    </div>
    <div class="side-column"></div>
</div>

Then we need to import the PostComponent in the timeline.component.ts file.

We'll wrap the post html in a container and give that a min width of 300px. 

# Angular to Rails API - Fetching posts from back end
We need to start the rails server: rails s

We'll set up environments in the client.
ng g environments

In the environment.development.ts file:

export const environment = {
    production: false,
    apiIrl: 'http://localhost:3000'
};

In the environment.ts file:

export const environment = {
    production: true,
    apiUrl: 'http://www.exampleproductionsite.com'
};

We'll create our post service:
ng g s core/services/post

In the post.service.ts file:
import...@Injectable({})...
import { Post } from '../../shared/models/post';
import { Observable } from 'rxjs';
import { HttpClient } from '@angular/common/http';
import { environment } from '../../../environments/environment';

export class PostService {
    constructor(private http:HttpClient) {}

    getTimelinePosts(): Observable<Post[]>{
        return this.http.get<Post[]>(`${environment.apiUrl}/posts`)
    }
}

We'll need to provide the HttpClient in the providers array (in the app.config.ts file):
providers: [provideTouter(routes), provideHttpClient()]

Then we run ng serve

In our timeline.component.ts file we can take out the hardcoded Posts and just include:
posts: Post[] = [];

We'll also want to add implements OnInit and add ngOnInit.
And inject the service.
constructor(private postService:PostService) {}

ngOnInit(): void {
    this.postService.getTimelinePosts().subscribe({
        next: (posts: Post[]) => {
            this.posts = posts;
        },
        error: (error:any) => {
            console.error('Error fetching timeline posts', error);
        }
    })
}

We get a CORS error.
In the Gemfile of the API we add gem "rack-cors"
Then run bundle install

In the config/initializers/cors.rb file:
Comment back in the Rails.application.config.middleware...
and change "example.com" to "*" in order to accept requests from anywhere
[we'll change this later]

At this stage we can comment out the before_action :authenticate_request because we don't have a way for a user to login at this point, so we get an unauthorized error.

At this point we can create posts in the database (using the rails console and Post.create, etc)

[Here we need to remove the title from the model in the frontend - there isn't a title for posts in the database]

# Rails seeds file - populate users and posts
We'll use the db/seeds.rb file in the API:
(1..10).each do |i|
    user = User.create(
        username: Faker::Internet.username(specifier: 3..20, separators: %w(_)),
        email: Faker::Internet.email,
        first_name: Faker::Name.first_name,
        last_name: Faker::Name.last_name,
        password: 'password',
        password_confirmation: 'password'
    )

    rand(1..10).times do
        user.posts.create(content: Faker::Lorem.paragraph)
    end
end

Then we run rails db:seed

Then we can run the console: rails console
and check User.count and Post.count
[in German's example there are 13 users and 51 posts]

Then when the frontend requests the posts in the ngOnInit it displays all the Faker generated posts.

# Fetching events with pagination

In the frontend project, in the styles.css (or scss):
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
}

ng g c shared/components/navigation --standalone

In the navigation.component.ts file:
import { RouterModule } from '@angular/router';

imports: [RouterModule],

In the navigation.component.html file:
<nav class="navbar">
    <div class="brand-name">Pickup Sports</div>
    <div class="nav-links">
        <a routerLink="" routerLinkActive="active" [routerLinkActiveOptions]="{exact:true}">TimeLine</a>
        <a routerLink="/events" routerLinkActive="active">Events</a>
    </div>
</nav>

In the navigation.component.scss file:
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #ADD8e6;
    padding: 0.5rem 1 rem;
    position: sticky;
    top: 0;
    width: 100%;
}

.navbar .brand-name {
    font-size: 1.5rem;
    font-weight: bold;
    color: white;
    padding-left: 0.5rem;
}

.navbar .nav-links a {
    color: #333;
    text-decoration: none;
    padding: 0.5rem 1rem;
    transition: background-color 0.3s ease-in-out;
}
.navbar .nav-links a:hover,
.navbar .nav-links a.active {
    color: white;
    background-color: #FFA500;
    border-radius: 4px;
}

In order to activate this, we need to go to the app.component.ts file and import the navigation component. 
Then at the top of the app.component.html we add:
<app-navigation />

Next we'll generate the events component:
ng g c features/events

Then in the app.routes.ts file, we'll add the path:
{
    path: 'events',
    loadComponent: () => import("./features/events/events.component").then((c) => c.EventsComponent)
}

In the api project (backend), we'll need to add a gem to the Gemfile:
gem 'kaminari'

This will allow us to paginate through the events; we can grab just 10 at a time, for example.
We can run rails s

Then in the events_controller.rb file:
before_action :authenticate_request, except: [:index]

(Because this project has authentication for this pages, we'll temporarily say not to worry about authentication for the index action.)

Then for the index action, we'll change Event.all:
def index
    events = Event.all
    render json: events, status: :ok
end

Instead we'll do:
def index
    events = Event.order(created_at: :desc).page(params[:page]).per(12)

    render json: {
        events: EventBlueprint.render_as_hash(events, view: :short),
        total_pages: events.total_pages,
        current_page: events.current_page
    }
end

Then in the app/blueprints/event_blueprint.rb file:
class EventBlueprint < Blueprinter::Base
    identifier :id

    view :profile do
        fields :content, :start_date_time, :end_date_time, :guests, :title
    end
end

We'll include another view:

view :short do
    fields :title, :start_date_time, :end_date_time, :guests, :sports
    association :user, blueprint: UserBlueprint, view: :normal
end

Then on the frontend we'll create a model for Event:
ng g class shared/models/event

In the app/shared/models/event.ts file:
import { User } from "./user";

export class Event {
    id: number;
    title: string;
    content: string;
    start_date_time: string;
    end_date_time: string
    created_at: string;
    user: User;

    constructor(event:any) {
        this.id = event.id || 0;
        this.title = event.title || "";
        this.content = event.content || "";
        this.start_date_time = event.start_date_time;
        this.end_date_time = event.end_date_time;
        this.created_at = event.created_at;
        this.user = event.user || new User({})
    }
}

Then we'll generate a service for our events:
ng g s core/services/event


In the app/core/services/event.service.ts file:
In the contructor we'll inject HttpClient.
We'll also import Event and environment.

getEvents(page:number){
    return this.http.get<Event[]>(`${environment.apiUrl}/events?page=${page}`)
}

Then in the app/features/events/events.component.ts file:
We'll implement OnInit.

currentPage: number = 1;
totalPages: number = 0;
events: Event[] = [];

constructor(private eventService:EventService) {}

ngOnInit(): void {
    this.loadEvents(this.currentPage)
}

loadEvents(page:number) {
    this.eventService.getEvents(page).subscribe({
        next: (response:any) => {
            this.events = response.events;
            this.currentPage = response.current_page;
            this.totalPages = response.total_pages;
        },
        error: (error:any) => {
            console.error("error fetching events", error)
        }
    })
}

nextPage() {
    if(this.currentPage < this.totalPages) {
        this.loadEvents(this.currentPage + 1)
    }
}

previousPage() {
    if(this.currentPage > 1) {
        this.loadEvents(this.currentPage - 1)
    }
}

In the events.component.html file:
@for (event of events; track event.id) {
    {{ event.title }}
}

<div class="pagination-controls">
    <button (click)="previousPage()" [disabled]="currentPage === 1">Previous</button>
    <button (click)="nextPage()" [disabled]="currentPage === totalPages || totalPages === 0">Next</button>
</div>

Then in the db/seeds.rb file we can add:

rand(1..10).times do
    user.created_events.create(
        title: Faker::Lorem.sentence,
        end_date_time: Faker::Time.forward(days: 25, period: :morning),
        start_date_time: Faker::Time.forward(days: 25, period: morning),
        guests: rand(1..10),
        content: Faker::Lorem.paragraph
    )
end

Then run rails db:seed

Then in the frontend we'll create and event component:
ng g c shared/components/events/event

In the event.component.ts file:
export class EventComponent {
    @Input({required: true}) event: Event = new Event({})
}

In the event.component.html file:
<div class="event-list-item">
    <div class="event-title">{{ event.title }}</div>
    <div class="event-username">{{ event.user?.username }}</div>
    <div class="event-datetime">
        {{ event.start_date_time | date }} - {{ event.end_date_time | date }}
    </div>

    <button class="view-more-button">View More</button>
</div>

We'll need to import the DatePipe in the ts file.

Then in the event.component.scss file:
.event_list_item {
    background-color: white;
    border: 1px solid #ddd;
    border-radius: 4px;
    padding: 1rem;
    margin-bottom: 1rem;
    transition: box-shadow 0.3s ease-in-out;
}

.event-list-item:hover {
    box-shadow: 0 4px 8px fgba(0,0,0,0.1);
}

.event-title {
    color: #333;
    font-size: 1.2rem;
    font-weight: bold;
    margin-bottom: 0.5rem;
}

.event-username {
    color: #666;
    font-size: 1rem;
    margin-bottom: 0.5rem;
}

.event-datetime {
    color: #555;
    font-size: 0.9rem;
    margin-bottom: 0.5rem;
}

.view-more-button {
    background-color: #FFA500;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 0.5rem 1rem;
    cursor: pointer;
    font-weight: bold;
}

Then in the events.component.ts file we need to import the event component.

Then instead of {{ event.title }} in teh events.component.html file we can use
<app-event [event]="event"/>

German updates the events.component.html file:

<div class="event-list-container">
    @for (event of events; track event.id) {
        <app-event [event]="event" />
    }
</div>

In the events.component.scss file:
.event-list-container {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 1rem;
    padding: 1rem;
    min-height: 90vh;
}

.pagination-controls {
    display: flex;
    justify-content: center;
    padding: 1rem;
}

.pagination-controls button {
    background-color: #ADD8E6;
    border: none;
    border-radius: 6px;
    padding: 0.5rem 1rem;
    margin: 0 0.5rem;
    cursor: pointer;
    font-weight: bold;
    transition: background-color 0.3s ease-in-out;
}

.pagination-controls button:hover {
    background-color: #FFA500;
    color: white;
}

.pagination-controls button:hover[disabled] {
    background-color: #ccc;
    cursor: default;
}

We want to keep track of the query params in the address bar.
In the events.component.ts file:
We'll include the router in the constructor.
private router:Router, private route:ActivatedRoute

Then in the ngOnInit we update it to:
this.route.queryParams.subscribe(params => {
    const page = params['page'] ? Number(params['page']) : 1
    this.loadEvents(page)
})

Then we update the nextPage and previousPage methods.
this.router.navigate([], {
    relativeTo: this.route,
    queryParams: { page: this.currentPage + 1 },
    queryParamsHandling: 'merge'
})

this.router.navigate([], {
    relativeTo: this.route,
    queryParams: { page: this.currentPage - 1 },
    queryParamsHandling: 'merge'
})

That updates the addresss bar to say localhost:4200/events?page=1 (for example)
