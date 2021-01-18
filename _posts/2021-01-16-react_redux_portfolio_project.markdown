---
layout: post
title:      "react redux portfolio project"
date:       2021-01-16 19:09:21 -0500
permalink:  react_redux_portfolio_project
---


![bebe](https://media.tenor.com/images/f24f34e1c1cf200df3c76111ef5972b0/tenor.gif)

## the plan
During this pandemic, many people have become new parents -- whether of new plants, new pets, or little humans. My project will be a daily tracker for these new parents to record the growth and care of their new little ones.

**User**
```
class User < ApplicationRecord
  has_secure_password
  validates :username, uniqueness: true
  has_many :bebes, dependent: :destroy
end
```

**Bebe**
```
class Bebe < ApplicationRecord
  belongs_to :user
  has_many :days, dependent: :destroy
  has_many :trackings, through: :days
  validates :name, presence: true
  validates :birthdate, presence: true
  validates :kind, inclusion: { in: %w(human plant pet),
    message: "%{value} is not a valid kind" }
end
```

**Day**
```
class Day < ApplicationRecord
  belongs_to :bebe
  validates :date, presence: true
  validates :date, uniqueness: { scope: :bebe_id,
    message: "track data daily per bebe" }

  has_many :trackings, dependent: :destroy
end
```

**Tracking**
```
class Tracking < ApplicationRecord
  belongs_to :day

  validates :info_type, presence: true
  validates :amount, presence: true
  validates :amount_unit, presence: true

  validates :info_type, inclusion: { in: %w(food water milk pee poop exercise sleep),
    message: "%{value} is not a valid tracked data point" }
end
```

## schema
```
ActiveRecord::Schema.define(version: 2021_01_03_185733) do

  # These are extensions that must be enabled in order to support this database
  enable_extension "plpgsql"

  create_table "bebes", force: :cascade do |t|
    t.string "name"
    t.date "birthdate"
    t.string "kind"
    t.text "bio"
    t.bigint "user_id"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.string "img"
    t.index ["user_id"], name: "index_bebes_on_user_id"
  end

  create_table "days", force: :cascade do |t|
    t.string "picture"
    t.date "date"
    t.text "note"
    t.bigint "bebe_id"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["bebe_id"], name: "index_days_on_bebe_id"
  end

  create_table "trackings", force: :cascade do |t|
    t.string "info_type"
    t.time "start_time"
    t.time "end_time"
    t.float "amount"
    t.string "amount_unit"
    t.text "note"
    t.bigint "day_id", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["day_id"], name: "index_trackings_on_day_id"
  end

  create_table "users", force: :cascade do |t|
    t.string "name"
    t.string "username"
    t.string "password_digest"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

  add_foreign_key "trackings", "days"
end

```

## the routes
1. Register/Login & Authenticate
2. Authorize => User Dashboard (See all Bebes)/Bebe Form
2. Authorize => Bebe Overview (See Bebe's Days)/Day Form
3. Authorize => Bebe's Day (See Bebe's Day's Trackings)/Tracking Form

Use of { Route, Link, BrowserRouter as Route, Redirect } from ‘react-router-dom’
I don't plan on having too many direct navlinks because they should be accessed via relationships, but I should have a home page which directs to a user's dashboard upon login, and a user should be able to hit some edit profile/settings link to edit their information. Aside from that, user's bebes will be accessed through 
i.e.
```
<Switch>
	<Route/>
	<Route/>
	<Route/>
</Switch>
```


## components
class components: class extends React.Component which must have a render() { return (JSX) } and have state, lifecycle methods, and can contain custom class methods.
pure components have state, lifecycle methods except for shouldComponentUpdate()
functional components: const X = (props) => {return (JSX)} are for when we don't need state or lifecycle methods

containers are components that render other components

## gist of lifecycle
i loved referring back to this diagram to understand the lifecycle: [https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/](http://)

Generally, it starts with a constructor which sets the initial state and binds any functions to this if needed, as well as extending functions with super(). If there is no need for state or binding functions, a constructor isn't necessarily needed. Additionally, with ES7, state can just be initialized within the component, without use of the constuctor.

The key in lifecycle sinking in for me was understanding/asking -- when do I want my component to re-render? I want it to render initially with any properties/initial state I give it, and re-render if there are any changes to props or state. 

So how do I make sure it knows state or props have changed? Through functions like setState({}). I can also call on componentDidUpdate() which gets invoked immediately after a component updates.

I didn't use componentWIllUnmount() in my project, but this would be where any clean-up occured if I had any running processes like timers.
## the class components
For this project, I did not use react hooks as not in the curriculum leading into this project, but react hooks would allow many functions that class components have but with a functional component!

Class components are useful when you require use of state, so my all my forms use class components as a "controlled component" (or a form that gets its inputs from state), as well as components that I wanted to use state to get creative in deciding how many items to show on the page using state.
1. User Registration Form
2. Login User Form
3. Create Bebe Form
4. Update/Edit Bebe Form
5. Bebe Data (Create Days)
6. Bebe Day (Create Trackings)
7. Bebe Days (use state to store/compare # vs # of days fetched to manipulate page display)

I also used class components for containers in which I wanted to make use of lifecycle functions, like performing fetch requests when a parent container loads in order to pass down data as props to children components.
1. Bebes Container (Displays Bebes and Bebe Form)
2. Bebe Container (Displays Bebe Data)


## the stateless components (functional)
these stateless components should not have local state, but may access state from the redux store via props passed down to the child component. 
1. NavBar to Display Routes/Links
2. Bebes (takes information from redux store and displays each Bebe as a card within the BebesContainer)
3. BebeInfo (takes in props from parent and displays Bebe's attributes)
4. Home Page (displays links and a simple photo)
5. Profile Page (displays user's info and relationships)

## the actions
The actions triggered in this application will include:
* CREATE_USER, LOGIN_USER
* FETCH_BEBES, CREATE_BEBE, EDIT_BEBE, FETCH_A_BEBE, DELETE_BEBE
* FETCH_BEBE_DAYS, CREATE_DAY

action creator/function creates an action object, and that action object is dispatched to the reducer which will change our state based on the action that was sent

Of note, when fetching data, due to asynchronous nature and return of a promise, a normal dispatch would not occur correctly with the connect(), so thunk allows the dispatch to be called at the end of the action i.e. the 2nd argument in connect(). The action function should return a function in which dispatch is passed in as an argument, the fetch request is run, the response is converted into a JSON object, and then that data returns the dispatch with a type and payload.
e.g.
```
export function fetchBebes() {
	return (dispatch) => {
		fetch(bebesURL)
		.then(resp => resp.json())
		.then(bebesData => dispatch({
			type: “FETCH_BEBES”,
			payload: bebesData
			})
		)
	}
}
```

I chose to keep Trackings local rather than store than in the redux store because no other component will really need access to the trackings except for the day page that will display them, thus I kept it local within the state for that component.

### [rails backend](https://github.com/beckyphan/bebe-backend)
rails new bebe-backend --api --database=postgresql --no-test-framework
(consider namespacing controllers into api/v1 to help with versioning organization in backend and use postgres for db)
I initially forgot to indicate the use of postgres upon creation, so had to go back and replace the db gem + update the database.yml file in the initializers/config folder.

* set up models
* set up controllers
* configure routes (namespace routes api/v1)
*e.g. rails g resource User username:string*
* create serializers
* create db/run migrations

A good resource for determing whether to generate just the model, resources, or scaffold: https://medium.com/@matt.readout/rails-generators-model-vs-resource-vs-scaffold-19d6e24168ee

One of the things I struggled with in the process with creating with javascript project with rails API backend was building vertically so it'll be good to continue practicing this! 

To start, I will need to build out the user model and making sure a user can register and login.
Then I should create the bebe model and ensure the relationships between a user and bebe are all working and that a logged-in user is able to create and view bebes.
Next, I will build the days model, the trackings model, and then ensuring everything works.

### [react frontend](https://github.com/beckyphan/bebe-frontend)
create-react-app bebe-frontend
* add gem 'rack-cors' and configure cors.rb file within initializers folder to allow cross-origin resource sharing
* add gem ‘active_model_serializer’ to be able to generate serializers via ‘rails g serializer’
Update attributes for each serializer & relationships


- src
- - App.js
- - index.js
- 
create & commit to github 
change App.js into class component

* install redux, react-redux, react-router-dom, redux-thunk —save

import into index.js file
- { createStore, applyMiddleware, compose } from ‘redux’
- thunk from ‘redux-thunk’
- { Provider } from ‘react-redux’
- reducer
- { BrowserRouter as Router } from ‘react-router-dom’

- set-up reducer
    - reducer is a function that takes in a state (set default as object) and an action
    - reducer will use switch/case to return an updated state based on what action was passed in
- create store in index.js

import into App.js file
- { connect } from ‘react-redux’
- { anyActions} } from actions folder to dispatch
- any container components
** remember to export default connect(mapStateToProps, mapDispatchToProps)(App)

connecting a function mapStateToProps allows us to access values in our store as props; 2nd argument in connect allows us to dispatch new actions directly from our component as props.

i.e. if you wanted to access this.props.accounts
then your function would be:
```
const mapStateToProps = (state) => {
	return {
		accounts: state.accounts
	}
}
```

### the process
To start, I set up my backend and frontend using generators and create-react-app.
After successfully creating user forms to log in and to register, as well as generating the models + the controller actions to handle storage of that data, the app should retain current_user information and hide links to sign-in or register, while now displaying the user's bebes/add new bebe form.

In the index.js file, I have my App component wrapped in the Router, which allows for different pages to be accessed. The entire thing is wrapped in the Provider which allows the App (and all components in it) to access the Redux store.

In my project, when a user logs in or registers, the user's info will be sent to the Redux store, where it is accessible and other components can check the store to see whether or not a user exists to authorize pages or to display the user's data. 

When a user is logged in, they'll be redirected to their Bebes page where they can see existing Bebes or add a new one. If they click into a Bebe, they'll be able to add days and add tracking data to the days, etc.

The last piece I wanted to do is keep a user logged in. As the user's info is held in the redux store and I am checking into the store to detect a user to authorize view of a page, the page will be inaccessible or will not load accurate if there is a refresh causing the store to disappear. I could use localStorage to hold a token with the user's information, however, the way my components are currently set up to pull information to display be accessing the store, I would also need access to other properties in my store. 

To do this, I used Redux Persist and updated my index.js file accordingly.

Future steps/stretch goals: 
* I want to implement in future versions of my project would be to display a calendar of days, rather than just created days. It would be visually appealing, as well as help with user experience in understanding which days contain data, etc. 
* I also want to be able to implement file uploads so users can upload pictures of their bebes directly.
* I have additional attributes for my days model that I hope to flesh out --> in a calendar view, it'd be awesome to be able to show a picture of the day that is saved with that day instance displayed out

![index](https://res.cloudinary.com/bphan/image/upload/v1610919651/Slide1_kwhdjs.jpg)
![app](https://res.cloudinary.com/bphan/image/upload/v1610919652/Slide2_eenlzp.jpg)
![routes](https://res.cloudinary.com/bphan/image/upload/v1610919652/Slide3_reypsz.jpg)
![bebescontainer](https://res.cloudinary.com/bphan/image/upload/v1610919651/Slide4_u2bqph.jpg)
![abebecontainer](https://res.cloudinary.com/bphan/image/upload/v1610919652/Slide5_ocysxp.jpg)


And last but not least, my video walk-through is [here](http://https://res.cloudinary.com/bphan/video/upload/v1611001175/reactredux_bebesproject_mqq3ft.mp4)

