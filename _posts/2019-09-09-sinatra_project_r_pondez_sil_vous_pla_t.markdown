---
layout: post
title:      "Sinatra Project: répondez s'il vous plaît"
date:       2019-09-10 01:54:27 +0000
permalink:  sinatra_project_r_pondez_sil_vous_pla_t
---


> making a list,
> checking it twice,
> we're tying the knot after May,
> so répondez s'il vous plaît.
> 


Learning Sinatra and ActiveRecord have been an amazing push forward into imagining the possibilites of what can be done with coding.

For my Sinatra Project, I created a typical wedding RSVP website. The intention is to have users (allcomers) be able to create an account and be able to confirm their RSVP (yes/no), choose a meal, and add a plus-one + denote their guest's meal.

Using [Corneal](https://thebrianemory.github.io/corneal/), I created the template for my Sinatra App--complete with the MCV, environment file, db/migrate, config.ru, and gemfile.

The gemfile requires several gems for the site to work properly:
```
gem 'sinatra'     #Sinatra is a domain-specific language written in Ruby and runs on the Rack web server interface
gem 'activerecord', '~> 4.2', '>= 4.2.6', :require => 'active_record' #ActiveRecord is the magic behind creating databases to store the information created on forms we create and allowing querying via SQL
gem 'sinatra-activerecord', :require => 'sinatra/activerecord' #extends Sinatra with ActiveRecord helper methods and Rake tasks
gem 'rake'
gem 'require_all'
gem 'sqlite3', '~> 1.3.6'
gem 'thin'
gem 'shotgun'
gem 'pry'
gem 'bcrypt' #allows salting of passwords for security
gem 'tux'
```

On the home page, visitors will be able to see all guests attending. 
Users, once logged in, can view their own guests and update any information they have created, as well as update their meal options for themselves and their guests. Users are currently set up with the following schema:
    t.string  "first_name"
    t.string  "last_name"
    t.string  "username"
    t.string  "email"
    t.string  "password_digest"
    t.string  "rsvp",            default: "Yes"
    t.integer "guest_limit",     default: 3

A user themself would be a guest IF they RSVP "Yes." They also have a plus one option, for a total guest limit of 2 (defaulted, unchangeable except by site admin essentially). Therefore, in my project, the 'user' show page and the 'guest' show page are essentially one & the same so I have merged their views.

If RSVP'ing "No", a user should not be able to pick a meal or have any guests. This would then delete the guests (including guest-1 belonging to the user which is self) and any meals associated with them. The view pages at this point are adjusted accordingly so only a user that has RSVP'ed "Yes" may select or update their meal, or add their plus-one.

To elevate the site aesthetics, I added a background image and updated some of the css as well.

My final product ends up looking like this:
![](https://drive.google.com/file/d/1mVnziJU-Z596N0vD0nM9DjC8Ry47i8Ay/preview)

In theory, only users who were invited to sign up should be signing up, but I did add a link so that users can delete their whole account, if they wish to.

Some future functionalities to consider:
- Seed Data of initial guest list with guest_limit of each set to the allowed # of plus-one's; users would basically be pre-signed up and need to complete their info to create an account.
- Users can update their email address/contact info & change password







