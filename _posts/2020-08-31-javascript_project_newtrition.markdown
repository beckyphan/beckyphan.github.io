---
layout: post
title:      "javascript project: newtrition"
date:       2020-08-31 00:02:49 -0400
permalink:  javascript_project_newtrition
---


>  just-in-time with first-class functions,
>  javascript allows multi-tasking with asynchronous executions

My javascript project is going to be a simplified nutrition tracker in v1.

## the models

A **user** profile will have a name, email (unique), password, and then targets for grams of carbohydrates, protein, and fat. A user will have many logs.

A **log** is a list of foods that a user consumes for the day. A log belongs to a user, has a date, and has many logged_foods/has many foods through logged_foods. It also has custom attributes to store a calculated list of nutrition totals, as well as an array of loggedFoods listed as string values for the frontend to access when created into an object.

The **food** table will contain the schema for foods, along with a quantity, unit, and then carb, protein, and fat content. To avoid duplicating the same foods as food instances, I want to use a join table to associate foods with logs. This food reference model will have many logged_foods. In this case, foods are currently created via seed data. In the future, I'd like to have an admin-level user be able to curate and add/update foods via the single-page app.

The join table **LogFood** will belong to a log and belong to a food, and have an attribute of quantity. I also provided a scope to be able to pull out all log_food entries for a particular user.

## the stories

A user can register for an account and log in after successful account creation. 

Upon log in, they will be able to generate a log for the day. A user may not make multiple logs for the same date, and managed by the log#controller -- if there is not an existing log for the day, it will be created. Otherwise, it will pull up the existing log for the selected date.

To this log, they will be able to add foods from a food list to keep track of foods consumed on that day. A user can also continue adding foods throughout the day as they munch on another handful of food, as well as delete the food entry from that log if added in error. Since foods are related to a log through the log_food table, the changes will actually be made through the log_food controller which will take in the current_user's id, the current log's id, and the user's input on quantity.

The food list will serve as the basic schema of food with nutrition values, and these foods will be viewable on a side panel where users can quickly add foods to their shown log.

The page will display a percentage of the carb, protein, and fat consumed during that day out of the user's goal carb, protein, and fat designated in their profile.

## the flow
Upon login or registering, a user will select the first date of the log they want to display. Depending on whether this log currently exists, the log will be created or shown. The column with the log will display log date, nutrition sums, and a list of foods added to the log. Another column will appear of the foods available to add to the log. Upon clicking "ADD" from the foods table, a user will be able to indicate the quantity of the food to add, and then add to their log. Their log will update with the added food, and nutrition summary will be updated accordingly. Similarly, the list of food items on the patient's log will have "X" buttons to remove the food from the current log. When removed, the log will update to remove the food item, as well as update the nutrition sums.

A user will be able to switch to create/show another log by selecting the "Change Log Button" at the bototm of the page. When this button is selected, column one (where the log displays), will change to a date-picker. To circumvent issues for a user to try to add a food from column two to a non-existent/non-displayed log -- the "ADD" button will be hidden. Upon date selection and a log displaying again in column 1, a user will be shown the "ADD" button to resume adding foods to the displayed log.

## the structure & gems
This project has a rails backend and a javascript frontend. We learned that it is good to have the backend and the frontend as separate repositories, which would allow for easy changes should the backend or frontend change in the future. We also learned that namespacing models in our backend would allow for easy updates should there be updates to versioning later on (i.e. api/v1).

I used the rails app generator to generate the basic rails app with a postgres database, and then the frontend simply consisted of js, css, and html files.

One of the gems that will allow the backend and frontend to work together is rack-cors. It is middleware that will allow rack-based apps to be compatible with cross-origin resource sharing. Enabling this gem AND ensuring the config/initializers cors.rb file is updated to allow the appropriate origin will be necessary.

The fast_jsonapi gem was used to quickly serialize my data.

## css
Incorporating some of the css and design aspects to the project is one of my favorite parts because it makes it so visually appealing to work with. That said, I do get nitpicky at times and run through multiple designs. Any elements that would require some changes via javascript through some action, I would be sure to incorporate an id or class to in order to easy traversing through the DOM to select that appropriate element to update.

## building vertically & using branches.
I had to wrap my mind around building vertically, because my mind gravitates towards horizontal builds: make all models, make all controllers, make all XYZ.

Detailing the steps helped with this, but to be able to test out the associations, I did have to ensure all the models were created at the very least and then build upon it. It wasn't intuitive to do a complete vertical build as I wouldn't be able to test out the features without its appropriate associations. I did a poor job of building vertically, but with additional planning for future projects this can be improved.

## learning goals & notes
### variables: let, const
For the most part, in my project I used const for pathnames and items that do not change. 
Variables that may change are declared with let. 
We learned not use var, which is a global variable, to avoid having global variables that may cause conflicts.

### data structures
There are 6+1 primitive values in javascript: bigInt, boolean, undefined, number, string, symbol, and null. The ways js organizes data is generally as objects and functions. 

### functions, arrow functions
Functions are a first-class objects and treated like any other variable in javascript, with the distinguishing factor being that they are able to be called.

anonymous functions don't have a name and can be an instantly invoked function expression that runs immediately after it is defined. They can also be assigned to a variable to store function's return value is stored.
```function () {}```

named functions are declared with a name and are not instantly invoked. 
```function name() {}```
To instantly invoke named functions, they must be written as an expression and called (extra pair of parentheses following).
```(function name() {})()```

Arrow functions are a feature introduced in ES6, in which the word function need not be written, and the return keyword + curly brackets can be omitted if it is in a single line. Arrow functions lack "this" and are not hoisted (aka must be defined before they are used).



### hoisting, context, scope, closures
For the most part, most of my javascript is within the 'DOMContentLoaded' event listener. Because variables are able to be seen within the scope of their function, if I needed to use the variable for another function I would pass it through as an argument so the new function would be able to access it. Variables that had more broad use I could declare as a const and place outside of the scope of any singular function. 

### ES6 syntax
E6 allows for new features in javascript, such as the use of let and const to declare variables, the introduction of arrow functions, classes, default parameters, and some other functions.

### js eventing
JS event listeners are straight-forward.
```document.addEventListener('DOMContentLoaded', () => { }```
I want to first make sure the content is loaded so we can start listening to clicks or submits done on the loaded content.
From there, it's a matter of making sure I'm adding the event listener on the correct element(s), and listening for the right event. The body will consist of the work to be done, whether that be various functions or different actions, etc.

### fetch requests
Fetch requests for pulling info are in a simple enough format.
``` 
fetch(loginPath)
      .then(resp => resp.json())
      .then(userObj => {
        displayUserProfile(userObj)
      })
```
I'm fetching  from a particular path which returns a promise containing the response, then we extract the json body content from that response, and then take that content to do some action.

To post or upload data, we have to incorporate another argument to the fetch request.
```
      fetch(logPath, {method: "POST", headers: {"Content-Type": "application/json"}, body: JSON.stringify(newLogData)})
      .then(resp => resp.json())
      .then(logObj => {
        displayUserLog(logObj)
      })
```
In addition to providing the path to which the data will post, I need to provide the method: "POST", metadata about the data I am sending (headers), and lastly the data I am sending is in the body in a readable format. The JSON.stringify() function will take my data and convert it accordingly.

The metadata can be encapsulated in an object and passed in as a variable, as well as the body data.

Generally, with working fetch requests, I'll be able to accomplish what I need in that 2nd .then(). 
.catch() is a way I can catch silent errors and handle unexpected events/do something with that error message. Adding a .ca


### rails as an api, rendering json
As opposed to sending out data viewable through our ERB, we are returning serialized json data and rendering that data into readable data on the SPA. 

### js object orientation, class, constructors, inheritance
For my project, my data existed on the backend with rails, and instances of these objects were simply created using a class constructor. There was no need for any use of inheritance to extend any features.

