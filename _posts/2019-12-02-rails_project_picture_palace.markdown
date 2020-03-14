---
layout: post
title:      "rails project: picture palace"
date:       2019-12-02 19:26:59 -0500
permalink:  rails_project_picture_palace
---


> ruby on rails has everything you need-
> it is organized programming logic patterned as MVC!

For my project, my goal is to create a Rails application that allows for social networking around movie-viewing.

Users should have the capability to create an account, and mark movies as seen if they have seen those movies.
For movies that users have seen, they can add their review + rating (0-5 stars).

Users can also create events around movies. These events will have a user as a host, and also have users as attendees.
In these events, users can add comments to communicate with other users attending the event.

Movies will have its own attributes, as well as be associated with events and ratings, as well as users through those events & ratings. 

Additionally, there are a plethora of helpful gems that have fleshed out capabilities that were recommended for use with our rails gem, including:
```
gem 'bootstrap-sass', '>= 3.4.1'  # styling extension of CSS3
gem 'bootstrap', '~> 4.3.1'       # most popular HTML, CSS, and JS framework
gem 'jquery-rails'                # javaScript library
gem 'devise'                      # registration/mailing/login, etc
gem 'cancancan'                   # user authorizations
gem 'simple_form'                 # form builder + compatibility with devise
gem 'rspec-rails'                 # for test building
gem 'omniauth-google-oauth2'      # for omniauth with google
```

The associations for this project were confusing for me, because I was going to have objects that had many of another object, but in two separate ways. I tried to wrap my head around this and read up more about activerecord associations via [the odin project](https://www.theodinproject.com/courses/ruby-on-rails/lessons/active-record-associations).

Mapping out the associations is incredibly helpful for me. It helps me see how the models are related, and prompts me to ask all the appropriate how's. For example, if a user has_many hosted events, but a user also has_many attended events--how does it know which is which? Which prompts me to go to the events model, where an event belongs to a host, but it doesn't contain a list of attendees. For that, I'll need to make a join table that lists the guest + the event so the info can be queried.

It was a bit of trial and error to get all the associations working the way I intended, and even then, I think there may be use of scopes that I'm unfamiliar with that can clean things up a bit. In the end, my models looked like this:

### User 
```
  has_many :hosted_events, foreign_key: :host_id, class_name: "Event"
  has_many :guestlists, foreign_key: :attendee_id
  has_many :attended_events, through: :guestlists, source: :event, class_name: "Event"

  has_many :movie_reviews, foreign_key: :reviewer_id, class_name: "Review"
  has_many :reviewed_movies, through: :movie_reviews, source: :movie, class_name: "Movie"

  has_many :to_watches
  has_many :movies_in_to_watches, through: :to_watches, source: :movie, class_name: "Movie"

  has_many :comments
```

```
  create_table "users", force: :cascade do |t|
    t.string "email", default: "", null: false
    t.string "encrypted_password", default: "", null: false
    t.string "reset_password_token"
    t.datetime "reset_password_sent_at"
    t.datetime "remember_created_at"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.string "username"
    t.index ["email"], name: "index_users_on_email", unique: true
    t.index ["reset_password_token"], name: "index_users_on_reset_password_token", unique: true
  end
```

### Movie
```
  has_many :reviews, dependent: :destroy
  has_many :events, dependent: :destroy
  has_many :reviewers, through: :reviews
```

```
  create_table "movies", force: :cascade do |t|
    t.string "name"
    t.string "genre"
    t.string "image_url"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end
```

### Event 
```
  belongs_to :host, class_name: "User"
  has_many :guestlists, foreign_key: :event_id, dependent: :destroy
  has_many :attendees, through: :guestlists, source: :attendee, dependent: :destroy

  has_many :comments, dependent: :destroy

  belongs_to :movie

  validates :host_id, uniqueness: { scope: :datetime, message: "you are already hosting another event at this date/time" }
```

```
  create_table "events", force: :cascade do |t|
    t.string "title"
    t.datetime "datetime"
    t.bigint "host_id", null: false
    t.bigint "movie_id", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.string "location"
    t.index ["host_id"], name: "index_events_on_host_id"
    t.index ["movie_id"], name: "index_events_on_movie_id"
  end
```

### Comment
```
  belongs_to :user
  belongs_to :event
```

```
create_table "comments", force: :cascade do |t|
    t.text "description"
    t.bigint "user_id", null: false
    t.bigint "event_id", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["event_id"], name: "index_comments_on_event_id"
    t.index ["user_id"], name: "index_comments_on_user_id"
  end
```

### Guestlist
```
  belongs_to :attendee, class_name: "User"
  belongs_to :event
```

```
  create_table "guestlists", force: :cascade do |t|
    t.bigint "attendee_id", null: false
    t.bigint "event_id", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["attendee_id"], name: "index_guestlists_on_attendee_id"
    t.index ["event_id"], name: "index_guestlists_on_event_id"
  end
```

### Review 
```
  belongs_to :reviewer, class_name: "User"
  belongs_to :movie

  validates :rating, numericality: {greater_than_or_equal_to: 0, less_than_or_equal_to: 5}
```

```
  create_table "reviews", force: :cascade do |t|
    t.integer "rating"
    t.bigint "reviewer_id", null: false
    t.bigint "movie_id", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.string "title"
    t.text "description"
    t.index ["movie_id"], name: "index_reviews_on_movie_id"
    t.index ["reviewer_id"], name: "index_reviews_on_reviewer_id"
  end
```

### ToWatch
```
  belongs_to :movie
  belongs_to :user

  scope :watched, -> { where(watched: true) }
  scope :tosee, -> { where(watched: false) }
```

```
  create_table "to_watches", force: :cascade do |t|
    t.boolean "watched"
    t.bigint "movie_id", null: false
    t.bigint "user_id", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["movie_id"], name: "index_to_watches_on_movie_id"
    t.index ["user_id"], name: "index_to_watches_on_user_id"
  end
```

In using devise, a lot of the user sign_up and log_in code are built in with the gem. In order to add a :username field, it was necessary to add the field in as a new migration to add_column. I updated the registration form in users/registration/new to include the :username field so that it would take up the field as a param. However, I was unable to figure out how to properly have the registration include the username.

I found a functioning workaround through updating routes.
```
devise_for :users, controllers: { registrations: 'users/registrations', omniauth_callbacks: 'users/omniauth_callbacks' }
```
registrations allowed for it to specify the controller being used for sign_up, and the omniauth_callbacks is for use with omniauth.

Then, I updated the sanitized params in the registrations controller to permit :username and added a @user.update into the user#create method in devise/registration to take in the :username.

There are a few specific views I want a user to be able to view, including:
- events#index
- movies#index
- user#show = dashboard; to prevent users from viewing other users' dashboards, instead of pulling from the params[:id], it is really calling upon the current_user method to pull data

Additionally, I've used control flow in the views to only allow users to edit events if they are the host of the event, and to only show delete buttons to remove themselves from attending events. Users are also not allowed to post reviewed unless they indicate that they have seent the movies.

The dependency: :destroy added to the various models ensure that if an event is deleted, all its related events and reviews are also deleted. Ideally, I also intended for users to be able to comment on events that they are attending, but if they choose to no longer attend, upon deleting themselves from the guestlist, all their comments on the events page will also be deleted. However, since the guestlist is a pure join_table, I'm unsure how the dependency works here, so I have a functional workaround in the guestlists_controller to check through and destory all comments by the user on the event on which they are no longer attending.

Using scopes on the ToWatch model was a very useful tool to quickly tease out the watched: true's vs the watched: false's
I am using it in the users#show controller to count up the movies that the user has marked as want to see vs has seen to provide a little blurb on the user's movie stats.

## Routes
```
  root to: "movies#home" 
#                                  root GET      /                                                                                        movies#home
#necessary when using devise
			 
  resources :guestlists
#                            guestlists GET      /guestlists(.:format)                                                                    guestlists#index
#                                       POST     /guestlists(.:format)                                                                    guestlists#create
#                         new_guestlist GET      /guestlists/new(.:format)                                                                guestlists#new
#                        edit_guestlist GET      /guestlists/:id/edit(.:format)                                                           guestlists#edit
#                             guestlist GET      /guestlists/:id(.:format)                                                                guestlists#show
#                                       PATCH    /guestlists/:id(.:format)                                                                guestlists#update
#                                       PUT      /guestlists/:id(.:format)                                                                guestlists#update
#                                       DELETE   /guestlists/:id(.:format)                                                                guestlists#destroy
 
 resources :comments
#                              comments GET      /comments(.:format)                                                                      comments#index
#                                       POST     /comments(.:format)                                                                      comments#create
#                           new_comment GET      /comments/new(.:format)                                                                  comments#new
#                          edit_comment GET      /comments/:id/edit(.:format)                                                             comments#edit
#                               comment GET      /comments/:id(.:format)                                                                  comments#show
#                                       PATCH    /comments/:id(.:format)                                                                  comments#update
#                                       PUT      /comments/:id(.:format)                                                                  comments#update
#                                       DELETE   /comments/:id(.:format)                                                                  comments#destroy

  resources :events
#                                events GET      /events(.:format)                                                                        events#index
#                            edit_event GET      /events/:id/edit(.:format)                                                               events#edit
#                                 event GET      /events/:id(.:format)                                                                    events#show
#                                       PATCH    /events/:id(.:format)                                                                    events#update
#                                       PUT      /events/:id(.:format)                                                                    events#update
#                                       DELETE   /events/:id(.:format)                                                                    events#destroy

  resources :movies
#                                movies GET      /movies(.:format)                                                                        movies#index
#                                       POST     /movies(.:format)                                                                        movies#create
#                             new_movie GET      /movies/new(.:format)                                                                    movies#new
#                            edit_movie GET      /movies/:id/edit(.:format)                                                               movies#edit
#                                 movie GET      /movies/:id(.:format)                                                                    movies#show
#                                       PATCH    /movies/:id(.:format)                                                                    movies#update
#                                       PUT      /movies/:id(.:format)                                                                    movies#update
#                                       DELETE   /movies/:id(.:format)                                                                    movies#destroy
	
  resources :reviews
#                               reviews GET      /reviews(.:format)                                                                       reviews#index
#                                       POST     /reviews(.:format)                                                                       reviews#create
#                            new_review GET      /reviews/new(.:format)                                                                   reviews#new
#                           edit_review GET      /reviews/:id/edit(.:format)                                                              reviews#edit
#                                review GET      /reviews/:id(.:format)                                                                   reviews#show
#                                       PATCH    /reviews/:id(.:format)                                                                   reviews#update
#                                       PUT      /reviews/:id(.:format)                                                                   reviews#update
#                                       DELETE   /reviews/:id(.:format)                                                                   reviews#destroy
	
  resources :to_watches, only: [:create, :update]
#                            to_watches POST     /to_watches(.:format)                                                                    to_watches#create
#                              to_watch PATCH    /to_watches/:id(.:format)                                                                to_watches#update
#                                       PUT      /to_watches/:id(.:format)                                                                to_watches#update

  resources :users, only: [:show]
#                                  user GET      /users/:id(.:format)                                                                     users#show

  resources :users, only: [:show] do
    resources :events, only: [:show]
  end
#                            user_event GET      /users/:user_id/events/:id(.:format)                                                     events#show
#                                       GET      /users/:id(.:format)                                                                     users#show

  resources :users, only: [:show] do
    resources :reviews, only: [:show]
  end
#                           user_review GET      /users/:user_id/reviews/:id(.:format)                                                    reviews#show
#                                       GET      /users/:id(.:format)                                                                     users#show

  resources :users, only: [:show] do
    resources :to_watches, only: [:show]
  end
#                         user_to_watch GET      /users/:user_id/to_watches/:id(.:format)                                                 to_watches#show
#                                       GET      /users/:id(.:format)                                                                     users#show

  resources :movies, only: [:show] do
    resources :reviews, only: [:new]
  end
#                      new_movie_review GET      /movies/:movie_id/reviews/new(.:format)                                                  reviews#new
#                                       GET      /movies/:id(.:format)                                                                    movies#show

  resources :movies, only: [:show] do
    resources :events, only: [:new]
  end
#                       new_movie_event GET      /movies/:movie_id/events/new(.:format)                                                   events#new
#                                       GET      /movies/:id(.:format)                                                                    movies#show

  devise_for :users, controllers: { registrations: 'users/registrations', omniauth_callbacks: 'users/omniauth_callbacks' }
#                      new_user_session GET      /users/sign_in(.:format)                                                                 devise/sessions#new
#                          user_session POST     /users/sign_in(.:format)                                                                 devise/sessions#create
#                  destroy_user_session DELETE   /users/sign_out(.:format)                                                                devise/sessions#destroy
# user_google_oauth2_omniauth_authorize GET|POST /users/auth/google_oauth2(.:format)                                                      users/omniauth_callbacks#passthru
#  user_google_oauth2_omniauth_callback GET|POST /users/auth/google_oauth2/callback(.:format)                                             users/omniauth_callbacks#google_oauth2
#                     new_user_password GET      /users/password/new(.:format)                                                            devise/passwords#new
#                    edit_user_password GET      /users/password/edit(.:format)                                                           devise/passwords#edit
#                         user_password PATCH    /users/password(.:format)                                                                devise/passwords#update
#                                       PUT      /users/password(.:format)                                                                devise/passwords#update
#                                       POST     /users/password(.:format)                                                                devise/passwords#create
#              cancel_user_registration GET      /users/cancel(.:format)                                                                  devise/registrations#cancel
#                 new_user_registration GET      /users/sign_up(.:format)                                                                 devise/registrations#new
#                edit_user_registration GET      /users/edit(.:format)                                                                    devise/registrations#edit
#                     user_registration PATCH    /users(.:format)                                                                         devise/registrations#update
#                                       PUT      /users(.:format)                                                                         devise/registrations#update
#                                       DELETE   /users(.:format)                                                                         devise/registrations#destroy
#                                       POST     /users(.:format)                                                                         devise/registrations#create
```

As seen, devise has a lot of built-in routes (and there are more as well with mailers, etc not shown here.
The updates I made included adding the :users scope. NOTE: need to update config/initializers/devise.rb file to allow the following:
```
config.scoped_views = true
 config.default_scope = :user
```

There are still a few things I am working to build out, including the user dashboard & routing + add a search bar/filtering tool to be able to highlight specific events that the user can specify e.g. events for specific movies, events that are in the future vs events that have already occurred.
~ circa Dec 2019


# UPDATE: MAR 2020
I took a really long hiatus between finishing up the rails project above & jumping in (between year-end holidays, winter break vacation, relocating from NYC to SoCal & allllll of the things that come along with making a cross-country move and establishing new residency, new job, new health networks, etc.), not to mention wedding planning. Whew.

One huge issue I was having when I last left off with using the devise gem is that I don't fully comprehend all the magic it contains. In trying to get my project to login with new attributes from the get-go or routing to a different page than default, it involves adjusting code that is pre-set and easy to use, but when it breaks, I cannot find where it goes wrong. When logged in and continuing to build out code, everything works that I intended to work... however, upon registration, I cannot determine the source of the break because I lack comprehensive knowledge of the devise tool.

With that said, I'm going to save what I have from this project and jump back into it a different time.

I am going to restart my project without using the devise gem, 1) to do a much needed refresher on my rails knowledge and 2) to intentionally create the login/logout processes from scratch to avoid the issues I'm running into with customization with the devise gem.

So here goes:

I am still creating a social application around movie viewing.

### User
A user will be able to register for an account & login with a password.

Any user will be able to create movies, 
a user can create events for movies, 
a user can comment on the events for movies, 
and a user can create reviews for movies.

### Movie
A movie can have multiple events,
and it can have many reviews.

### Review 
A review is written by a user for a movie

### Comment 
A comment is written by user for an event of a movie

### Guestlist
A guestlist is the join table between a user who is attending an event and an event.

## Aliasing
I have a lot of trouble understanding how to properly alias in rails, but in my model, there are going to be some items that have a dual association so I have to rename the attribute in order to approrpriately pull the correct info.

Each user can create multiple events, aka they can host multiple events.
Each user can also attend multiple events, aka they are attendees of multiple events.

Has_many events won't be enough to differentiate the two, so I have used aliasing.

In this way, my models & migrations are as follows for the specific attributes:

```
class User < ApplicationRecord
  has_many :events, foreign_key: 'host_id' 
	# event.host will know that host == user and pull the corresponding user object
  # foreign_key is referencing what column to look for in the Event table
	
	has_many :guestlists
  has_many :events, through: :guestlists, foreign_key: 'attendee_id' 
	# event.attendee will know that attendee == user and pull the corresponding user object
  # foreign_key is referencing what column to look for in the Guestlist table

```

```
class Event < ApplicationRecord
 
  belongs_to :host, class_name: "User"
	# event.host will pull the host (aka user) who created the event

  has_many :guestlists
  has_many :attendees, through: :guestlists, foreign_key: "attendee_id"
	# event.attendees will pull the attendees (aka users) who are attending the event
  # foreign_key is referencing what column to look for in the Guestlist table
end

```

```
class Guestlist < ApplicationRecord
  belongs_to :event
  belongs_to :attendee, class_name: "User"
	# user.events will pull events that the attendee (aka user) is attending
end
```

To create working migrations, I ran into errors when trying to use the belongs_to because there are additional checks rails seems to be doing to ensure referential integrity, so the way to go was to explicity note the foreign keys in my migrations.

There is nothing special with the users able/users migration, because the aliasing has really occurred with its associated objects, i.e.  the users table will not need any special columns.

Rather, the affected tables are:
Events table, where a user is referenced to as a host - (host_id)
Guestlist (join table), where a user is referenced to as an attendee - (attendee_id)

```
MIGRATION
class CreateEvents < ActiveRecord::Migration[6.0]
  def change
    create_table :events do |t|
      t.belongs_to :movie, null: false, foreign_key: true
      t.integer :host_id, null: false
    end
  end
end


SCHEMA
create_table "events", force: :cascade do |t|
    t.bigint "movie_id", null: false
    t.integer "host_id", null: false
    t.index ["movie_id"], name: "index_events_on_movie_id"
  end
```


```
MIGRATION
class CreateGuestlists < ActiveRecord::Migration[6.0]
  def change
    create_table :guestlists do |t|
      t.belongs_to :event, null: false, foreign_key: true
      t.integer :attendee_id, null: false, foreign_key: true
    end
  end
end

SCHEMA
  create_table "guestlists", force: :cascade do |t|
    t.bigint "event_id", null: false
    t.integer "attendee_id", null: false
    t.index ["event_id"], name: "index_guestlists_on_event_id"
  end

```

## Scopes
Reading about scopes, documentation notes that it really just is 'syntactic sugar'  so when I got confused by its syntax, I just started off writing it as a class method, and then copying & pasting the method body into the scope template.

Additionally, I had to wrap my head around what sql queries I was planning to use.

For example, I want to know which event has the most attendees.
Essentially, rails would have to go through each event from Event.all and let me know how many attendees were in each; then it would need to order it by the attendee counts and return the top value.

In my head, it made sense to try to do Event.attendees, and call a count on that... 
but the class Event does not have attendees... an instance of an event has attendees. 

So in reality, I need to do a scope on my JOIN TABLE for Events & Attendees, which is my Guestlist table, whose attributes are attendee_id and event_id.

To get my guest counts, I GROUP the guestlist by :event_id and then call COUNT

```
class Guestlist < ApplicationRecord
  scope :event_guest_counts, -> { self.group(:event_id).count }
```

The SQL query would look like:
```
 SELECT COUNT(*) AS count_all, "guestlists"."event_id" 
 AS guestlists_event_id 
 FROM "guestlists" 
 GROUP BY "guestlists"."event_id"
 
 => {1=>1, 2=>5} 
```

It returns me the :event_id and its count in a hash for each event I have. 
i.e. For event with event_id: 1, I have 1 attendee & for event with event_id: 2, I have 5 attendees

Recall that Guestlist.event_guest_counts => {1=>1, 2=>5}  with key:value pairs being :event_id and then its count.
My order_events_by_popularity scope is a scope that orders the event_id listings by the attendee counts.

```
  scope :order_events_by_popularity, -> { self.group(:event_id).order("count(*)").count }
```

To make it easy to call upon, I created a class method to look up the event and pull out that object.

```
class Guestlist < ApplicationRecord
  def self.most_popular_event
    event_id = self.order_events_by_popularity.first.first
    @event = Event.find_by(id: event_id)
  end
end
```

## Nested Resources
In creating a movie review or in posting a comment on an event, a user really only has that action upon seeing a movie or viewing an event. 

In other words, when viewing movie reviews, I don't intend for users to be able to see a page with all the reviews ever written for all the movies in the database; it would always be associated with a movie.

First, I created my nested resources in my routes file.

```
  resources :movies do
    resources :reviews
  end
```
	
To make things very user-friendly, I created a review form on the movie#show page.

```
views/movie#show

<%= form_with model: @review, url: movie_reviews_path(@movie, @review), class: "review-form" do |f| %>
      <%= f.label :review_title %>
      <%= f.text_field :review_title><br>
      <%= f.label :rating %>
      <%= f.select :rating, [1, 2, 3, 4, 5] %><br>
      <%= f.text_area :description %>
      <center>
      <%= f.submit "Add My Review"%>
      </center>
    <% end %>
	```
	
Note the path for this form is movie_reviews_path(@movie, @review), where @movie is the current movie whose show page we are on which is noted in the movie#show controller, and @review = Review.new.

In the Review controller:
```
  def create
     @review = Review.new(review_params)
     @review.movie_id = params[:movie_id]
  end
	
	def review_params
    params.require(:review).permit(:review_title, :rating, :description)
  end
```

Params sent look like this:
<ActionController::Parameters {"authenticity_token"=>"XXX", "review"=>{"review_title"=>"New Joker Review", "rating"=>"4", "description"=>"Great Movie."}, "commit"=>"Add My Review", "controller"=>"reviews", "action"=>"create", "movie_id"=>"1"} permitted: false>

All the form elements are within the REVIEW hash, while the movie_id param which is taken from the movie_path url (/movies/:id(.:format)) is passed along separately, as a result of having a nested resource.

## OmniAuth
to be continued!


	
