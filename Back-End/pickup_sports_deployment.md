# Deploying a Rails API to Render Part 1 - Setup
First we need to change the type of database we're using.

We'll move the gem from the main area to group :development, :test
gem "sqlite3", "~> 1.4"

Then we'll add:
group :production do
    gem 'pg'
end

(pg is for postgresql)

Then in the config/database.yml file:
production:
    adapter: postgresql
    encoding: unicode
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
    url: <%= ENV['DATABASE_URL'] %>

Then in the root of the project we create a render.yaml file (this is specifically to use Render):
databases:
    - name: renderdeploymentexample
      databaseName: renderdeploymentexample
      user: renderdeploymentexample
      plan: free

services:
    - type: web
      name: renderdeploymentexample
      runtime: ruby
      plan: free
      buildCommand: './bin/render-build.sh'
      # preDeployCommand: "./bin/rails db:migrate" # preDeployCommand only available on paid instance types
      startCommand: './bin/rails server'
      envVars:
          - key: DATABASE_URL
            fromDatabase:
                name: renderdeploymentexample
                property: connectionString
          - key: RAILS_MASTER_KEY
            sync: false
          - key: WEB_CONCURRENCY
            value: 2 # sensible default

The name needs to be unique to the project: pickupsportsapi

In the bin folder we create a new file render-build.sh:
set -o errexit
bundle install
bundle exec rake db:migrate

Then we commit and push to GitHub.


# Deploying a Rails API to Render Part 2
Signup/sign-in to Render
(Easy to go through GitHub)

Because we're using a free platform, we can only use one deployed service.

Click on Blueprints
Then click on New Blueprint Instance
Then we search for the repository we want and click Connect
We give it a Blueprint name (Default Rails Render)
We have to enter the masterkey - we get that from the config/master.key file in the API
Click Apply

[German had an error - can click on deploy logs to see the error - it gives some code to run in the command line to support the platform. Then commit and push again.]

There will be a url for the project, but if we click on it, we'll get a 404.
The app is running, but we don't have a root route.
We can adjust the url to pickupsportsapi.onrender.com/posts
Then we'll see an empty array of posts. 

We can also test the endpoints in postman (for example, https://pickupsportsapi.onrender.com/users we can send a POST request to create a user).

[We'll want to update the blueprint so we don't get the password_digest hash back.]


# Deploying the Frontend to Vercel
We can create an account with Vercel using GitHub.

Click on Import Project
Find the repository we want and click Import
Then on the Configure Project page we click Deploy
(We'll wait for it to build, install dependencies, etc)
Click Continue to Dashboard
Click on Visit and it will open in the browser
pickup-sports-client.vercel.app

German tries to login, but then it gets an error.
We need to update the environment.ts file (the one with production: true,):
apiUrl: 'https://pickupsportsapi.onrender.com'

Then we need to commit and push the API again.
