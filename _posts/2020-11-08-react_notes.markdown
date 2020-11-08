---
layout: post
title:      "react notes"
date:       2020-11-08 23:08:13 +0000
permalink:  react_notes
---


## learning objectives and notes
### background
* react framework is built out of Javascript
* JSX (extension of JavaScript with a specific syntax) is separated out into reusable, indpendent components, and when combined, form a web application

React has a virtual DOM, declarative writing structure, a transpiler (Babel), a bundler (Webpack), and built-in ESLint functionality.

React apps require a server to the app to run on, accomplished by `npm start`

#### react components
components are independent, reusable pieces of code that can be thought of as little packages or JavaScript functions, accepting "props" (properties) and return React elements.

React components can be used as dynamic templates, and passed in "**props**" from parent components to children components to use. Of note, calling a component within itself can result in endless recursion if there are no break conditions.

to start, you generally need:
```import React from 'react'``` in your main file, as well as have
```import React, { Component } from 'react'``` in your component file. Also remember to ```extend Component``` when creating the class to be able to inherit those component features.

To avoid breakage when using props, default props may be used by adding ```.defaultProps``` to the class.

When rendering multiple components, collections of elements can be built and included using {} and looping through the array using the map() function. Keys are included to help React identify which items have changed/been added/been removed. Generally, IDs are used as keys, or if necessary, use the index. When using the index, be sure to have the index as an argument in your map function ```.map( (item, index) => ```

while props are limited due to needing parent component to send it props/new props, **state** provides a mthod to maintain/update information within a component without requiring a parent component to send updated info.

> to use states, use a constructor() to set the initial ```this.state = {}``` because constructor() runs first; and then create other methods that function to set the new state via ```this.setState({})```. setState() is asynchronous. The magic of .setState({}) is that it is a function available to all React components and informs that it should re-render. To update a state that references itself, the function previousState can be used such that ```this.setState(previousState => { return { attribute: previousState.attribute + action } })```


#### accessing components
Using children components in the top-most App component requires importing them into the App.js file, as well as exporting them in each respective component.

```export default class XComponent``` would import the entire component, so in App.js you can import ```import XComponent from [fileLocation/fileName].js```
vs
```export class YComponent``` would require you to name the specific class you want to import ```import { YComponent } from [fileLocation/fileName].js```


```export default [functionName]``` can only be used once per module to export a function.
similarly, ```export default class X extends React.component``` is used to export a component. sometimes, the component is written as a class to begin with, with an ```export default [className]``` at the end of the file, which can help with reading code/locating what is exported.
exporting default allows aliasing upon import.

```import * from [fileLocation/fileName].js``` indicates importing everything from that file *that is exported*.

named exports allow export of several specific things at once; Modules can be renamed upon import "as XYZ" and then referenced as XYZ.functionName() or can simply be imported normally and referenced by name.

callbacks are used to effect change outside of the context of a parent.

#### JSX
Functions can be called and must return a single JSX element or multiple JSX elements in an array.

Any valid Javascript expression can be placed within curly braces and used or embedded in an attribute.

#### Events
HTML events are utilized in React by wrapping them in a "SyntheticEvent"
the action is wrapped inside a function (usually an arrow function to avoid creating a new scope with different value of ```this```), and the event handler is attached to the element e.g. ```onClick={this.functionName}```

Event pooling is a technique that sends the event data object to the callback so that the object is cleaned up for later use. this prevents asynchronous access to data, unless the target is saved into a variable or the event is made persistent via event.persist()

#### Controlled vs Uncontrolled Components
Controlled vs uncontrolled components refer to the ways we can implent forms in react.
With controlled components, form data is handled by a React component so that form elements (such as inputs) are constantly saved in the react state to serve as the “single source of truth.
With uncontrolled components, the form sort of has its own state, and data is handled via the DOM.

#### React Component Lifecycles


#### React Containers
Presentational components or "simple components" are components that have no state and are also not affected by state. They simple render themselves or use props data. Uncoupling the logic piece from the presentation layer allows reuse of the presentation container to build a difference piece of UI with the same code by importing and wrapping it.



