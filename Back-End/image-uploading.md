# Intro to images
-comes with its own challenges and considerations
-requires handling file uploads, processing images, managing storage
-also requires ensuring sevurity and performance, and optimizing images for web use
-also need to take into consideration bandwidth usage (large images can slow down an app and consume more resources)

-file handling
Involves working with the file system to mangage files and directories. 
Includes reading, writing, updating, and deleting files.
Includes working with libraries like Active Storage for file operations and understading paths and buffers.

-image processing
Involves receiving, modifying, and storing images.
Includes resizing, formatting, and optimizing for web use.

-active storage
A built-in library in Ruby on Rails that provides a way to upload files to a cloud storage service like Amazon S3 or Google Cloud Storage. 
It also provides a way to attach files to Active Record models (which makes it easy to manage file uploads).

-cloud storage
Services like Amazon S3 and Google Cloud Storage store and manage files in the cloud.
Offered features: scalability, security, reliability.

Image Optimization
-reducing the file size of an image without significantly affecting its visual quality
Techniques: compression, resizing, format conversion
Different use cases: creating thumbnails, avatars, images for different screen sizes


# Active Storage in Ruby on Rails
A library that provides a way to upload images and attach them to Active Record models. Also provides features like image resizing and optimization.

-image resizing
Useful for creating different versions of an image for different use cases

-image optimization
Compressing images

-image cropping
Can crop images to a specific size

-image conversion
A way to convert images to different formats.

[here we'll add on to the project from the prevous lesson on deployment: render_deployment_example]

We'll first add Active Storage to our app:
rails active_storage:install

This creates a migration file that wil add the necessary files to the database.
# This migration comes from active_storage (originally 20170806125915)
class CreateActiveStorageTables < ActiveRecord::Migration[7.0]
  def change
    # Use Active Record's configured type for primary and foreign keys
    primary_key_type, foreign_key_type = primary_and_foreign_key_types

    create_table :active_storage_blobs, id: primary_key_type do |t|
      t.string   :key,          null: false
      t.string   :filename,     null: false
      t.string   :content_type
      t.text     :metadata
      t.string   :service_name, null: false
      t.bigint   :byte_size,    null: false
      t.string   :checksum

      if connection.supports_datetime_with_precision?
        t.datetime :created_at, precision: 6, null: false
      else
        t.datetime :created_at, null: false
      end

      t.index [ :key ], unique: true
    end

    create_table :active_storage_attachments, id: primary_key_type do |t|
      t.string     :name,     null: false
      t.references :record,   null: false, polymorphic: true, index: false, type: foreign_key_type
      t.references :blob,     null: false, type: foreign_key_type

      if connection.supports_datetime_with_precision?
        t.datetime :created_at, precision: 6, null: false
      else
        t.datetime :created_at, null: false
      end

      t.index [ :record_type, :record_id, :name, :blob_id ], name: :index_active_storage_attachments_uniqueness, unique: true
      t.foreign_key :active_storage_blobs, column: :blob_id
    end

    create_table :active_storage_variant_records, id: primary_key_type do |t|
      t.belongs_to :blob, null: false, index: false, type: foreign_key_type
      t.string :variation_digest, null: false

      t.index [ :blob_id, :variation_digest ], name: :index_active_storage_variant_records_uniqueness, unique: true
      t.foreign_key :active_storage_blobs, column: :blob_id
    end
  end

  private
    def primary_and_foreign_key_types
      config = Rails.configuration.generators
      setting = config.options[config.orm][:primary_key_type]
      primary_key_type = setting || :primary_key
      foreign_key_type = setting || :bigint
      [primary_key_type, foreign_key_type]
    end
end

This migration creates three tables:
active_storage_blobs - store info about the files that are uploaded
active_storage_attachments - used to associate files with Active Record models
active_storage_variant_records - used to store info about image variants that are created using Active Storage

Then we run rails db:migrate

In the User model we add:
class User < ApplicationRecord
  has_one_attached :image
end

We can now upload an image and associate it with a user.

We could attach many images to a user, for example:
class User < ApplicationRecord
  has_one_attached :avatar
  has_one_attached :cover_photo
  has_many_attached :images
end

We can test out our has_one_attached :image in the rails console

rails c

user = User.create(username: "test")
user.image.attached? # false

Then we'll create a route to upload an image.

In the config/routes.rb file:
Rails.application.routes.draw do
  resources :users do
    post 'upload_image', to: 'users#upload_image'
  end
end

And we'll add to the users_controller:
def upload_image
    user = User.find(params[:user_id])

    if user.image.attach(params[:image])
      render json: { message: "Image uploaded" }, status: :ok

    else
      render json: { message: "Image upload failed" }, status: :unprocessable_entity
    end
  end

Then we can test in Postman.
We'll attach an image and send a POST request to 
http://localhost:3000/users/1/upload_image
with the iamge as form data.

Click the body tab
Choose form-data
Add a key of image and change the type to file
Choose a file to upload
Click send

If successful, we should see a reponse of {"message": "Image uploaded"}

Then when we check user.image.attached? in the rails console we'll get true.
[I get a successful response ("Image uploaded"), but then user.image.attached returns false...]


Send image URL in response
We can use the rails_blob_path helper method.
In the User Controller:
  def upload_image
    user = User.find(params[:user_id])

    if params[:image] && user.image.attach(params[:image])
      render json: { message: "Image uploaded", url: rails_blob_url(user.image, only_path: false) }, status: :ok
    else
      render json: { message: "Image upload failed" }, status: :unprocessable_entity
    end
  end
end

Then an example of a response:
{
	"message": "Image uploaded",
	"url": "http://localhost:3000/rails/active_storage/blobs/redirect/eyJfcmFpbHMiOnsiZGF0YSI6MTEsInB1ciI6ImJsb2JfaWQifX0=--d90b205184baccb59c2d5613796003bbb9d70efb/angular-logo-1200-628.png"
}

[I tried with the new code for the rails_blob_path helper and I get an error - it won't send]


# Uploading images with Cloudinary
We'll use Active Storage locally, but Cloudinary in deployment. 
Sign up for a free account: 
https://cloudinary.com/users/register_free

Then we need to add the gem in our Gemfile in the production group:
group :production do
  gem 'pg'
  gem 'cloudinary'
end

Then run bundle install

We need to configure the cloudinary gem by adding the following to the 
config/environments/production.rb file:
require "active_support/core_ext/integer/time"

Rails.application.configure do
  # Settings specified here will take precedence over those in config/application.rb.
...
  config.active_storage.service = :cloudinary
...

We replace config.active_storage.service = :local 
with
config.active_storage.service = :cloudinary


# Credentials and Storage Configuration
Then we need to add the cloudinary configuration to storage.yml (used to configure the storage services that are used by Active Storage):
test:
    service: Disk
    root: <%= Rails.root.join("tmp/storage") %>

local:
    service: Disk
    root: <%= Rails.root.join("storage") %>

We'll need to add the cloud_name, api_key, and api_secret to the storage.yml file.
We get those from Cloudinary.

On the Cloudinary Dashboard we should see 
cloud_name
api_key
api_secret

Then we add them to the storage.yml file:
test:
    service: Disk
    root: <%= Rails.root.join("tmp/storage") %>

local:
    service: Disk
    root: <%= Rails.root.join("storage") %>

cloudinary:
    service: Cloudinary
    cloud_name: 'djlttsuf3'
    api_key: '1165244111841913'
    api_secret: 'pJPGYiJv3fhIC1ezruSQWjbkOJjk'

However, there is a problem - the cloud_name, api_key, and api_secret are exposed.
We can fix this with the credentials file to store the cloudinary configuration.

To edit the credential file, we run
EDITOR="code --wait" bin/rails credentials:edit

This will open a new file in the code editor. 
[Windows users may run into an issue with EDITOR. If that's the case, run rails credentials:edit and open the file manually.]

We can do this because we have the master key in our project folder.
It's used to decrypt the credentials.yml.enc file.

[If we need to have someone else have access to the master key we'll have to send that by another means - it won't be pushed to the repository.]

Here we'll store sensitive info:
# aws:
#   access_key_id: 123
#   secret_access_key: 345

cloudinary:
    cloud_name: 'djlttsuf3'
    api_key: '1165244111841913'
    api_secret: 'pJPGYiJv3fhIC1ezruSQWjbkOJjk'

# Used as the base secret for all MessageVerifiers in Rails, including the one protecting cookies.
secret_key_base: 908a3dca583e7754b741df137fb9a3b2c756c47448a349eacadfc4202c38f8682e5e3753f0dc858c22f7536e5ead6934ac5f8be3c48ca62f268a348ed409fdae

Then we close the file to save it.

We can access the cloudinary configuration in our storage.yml file and update:
test:
    service: Disk
    root: <%= Rails.root.join("tmp/storage") %>

local:
    service: Disk
    root: <%= Rails.root.join("storage") %>

cloudinary:
    service: Cloudinary
    cloud_name: <%= Rails.application.credentials.cloudinary[:cloud_name] %>
    api_key: <%= Rails.application.credentials.cloudinary[:api_key] %>
    api_secret: <%= Rails.application.credentials.cloudinary[:api_secret] %>

We access the values from the credentials.yml.enc file with the Rails.application.credentials method followed by the key and value we want to acess.

Then we can push to the repository and test this in production.

We can create a user and then use postman to send a POST request to the appname.com/users/1/upload_image with the image as a form-data.

We can also view the uploaded images in the Media Library in Cloudinary.


# notes from class 3/7/2024
has_one_attached :cover_image

Can have many images:
has_many_attched
[we want to be careful with that, though, for size and storage]
[we probably want to also include validations to make sure it's an image file type]

German didn't create a method for uploading the image

Add :cover_image to the blog_params (in the blogs_controller)

We can have a route with the endpoint /upload_image, but we can also just send a post request to the /blogs endpoint. Then in the form-data we also include all the other key/value pairs. (In postman).
Then when we create a new blog, we'll do all the key/values plus the key :cover_image and type file with the image we want to upload.

We'll use url_helpers to get the url of the image back in the response

In the Blog model (blog.rb), we can define a method cover_image_url (then we'll use that method in the blurprinter):

class Blog < ApplicationRecord
include Rails.application.routes.url_helpers

has_one_attached :cover_image
...
def cover_image_url
  rails_blob_url(self.cover_image, only_path: false) if self.cover_image.attached?
end

[this will give us the full image path]
[we don't have to include self, but it makes it clearer and more readable]

Then in the blogs_controller, in the show method we want to use the Blueprint.
render json: BlogBluepint.render(blogs, view ...)

class BlogBlueprint
...
fields :title, :content, :cover_image_url

In the environments folder, in development.rb (we'll set up the host):

Rails.application.configure do
  Rails.application.routes.default_url_option[:host] = "localhost:3000"
...
end

And we'll use a different host url for production.

In the frontend, we can create a request that's form data and attach the image.

We'll want to add the cover image to the Blog model in the frontend:
...
export class Blog {
...
cover_image_url: string = ""
...
}

In the HTML form (for a new blog), we'll add:
<div>
  <label for="cover_image">Cover Image</label>
  <input type="file" (change)="onFileSelected($event)" id="coverImage">
</div>

We use the $event to be made aware of when there's a change. The event object will include the file the user has uploaded.

Then in the TS file:

selectedFile: File | null = null;
...
onFileSelected(event:any) {
  if(event.target.files && event.target.files[0]) {
    this.selectedFile = event.target.files[0]
  }
}

Then we change the createBlog method (instead of just using the this.blogForm because it won't include the file)
{selectedFile: this.selectedFile, ...this.blogForm.value} - this won't work because we can't use raw. We need to use form-data.

createBlog() {
const formData = new FormData();
const title = this.blogForm.get['title']!.value;
const content = this.blogForm.get['content']!.value;

formData.append('title', title);
formData.append('content', content);
if (this.selectedFile) {
  formData.append('cover_image', this.selectedFile, this.selectedFile.name);
}

this.BlogService
  .createBlog(formData)
...
}

We also need to update the createBlog method in the service (with the corrent type):

Then we replace the hard-coded image in the html with the dynamic image:
[src]="blog.cover_image_url || other url"


The setup for production is different because we'll use a cloud service.
[do more reading on Rails Guides active storage]
Setup an account with Cloudinary.

In the Gemfile:
(In the production group)
gem 'cloudinary'

In the config/environments/production.rb file:
config.active_storage.service = :cloudinary

In the storage.yml file:
We'll need the api key, secret, and cloud_name

cloudinary:
  service: :cloudinary
  cloud_name: 'asldkfj'
  api_key: '239487'
  api_secret: '398ouskjh345'

However, we don't want to have these exposed in our files.

We'll use the credentials.yml.enc file
Because we have the master.key we can run a command to open the credentials file:
EDITOR="code --wait" bin/rails credentials:edit

We can't save the file with ctrl C. We have to close it manually by clicking the X
Then we paste the sensitive info in the credentials file.
And then use Rails.application.credentials.cloudinary[:cloud_name]
Rails.application.credentials.cloudinary[:api_key]
Rails.application.credentials.cloudinary[:api_secret]

We need to wrap them with tags:
<%=     %>


Then with those changes we'll need to push to GitHub (since this is for production).
And then check on Render that a new deployment has been triggered.

[German got an error because we didn't add the host url to production:
production.rb ... [:host]...]