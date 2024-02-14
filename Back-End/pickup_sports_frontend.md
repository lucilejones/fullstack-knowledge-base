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

Then when the fronend requests the posts in the ngOnInit it displays all the Faker generated posts.

# Fetchin events with pagination

