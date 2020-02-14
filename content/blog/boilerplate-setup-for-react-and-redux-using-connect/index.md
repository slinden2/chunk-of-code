---
title: "Boilerplate setup for React and Redux using connect"
published: true
date: "20190515"
tags: ["react", "redux"]
---

Setting up a new project can be a long process sometimes and usually requires a few web searches. You usually know what needs to be done, but you may not remember how exactly it must be done and which lines of code are necessary.

I started an example project with `create-react-app` and I made the basic file structure for `react-redux` project. This is just a starting point. Of course, as your projects goes on you will probably have a decent amount of files more than in my boilerplate example. You will probably also need some other libraries in your project, but for the sake of this example, I will consider only React and Redux.

## Intalling packages

First let's install the necessary packages. Remember that as a starting poit you should have a clean `create-react-app` project. Delete also everything that you have in the _src_ folder.

```
npm install redux react-redux redux-thunk redux-devtools-extension --save
```

- [redux](https://www.npmjs.com/package/redux): The state container itself
- [react-redux](https://www.npmjs.com/package/react-redux): Bindings between React and Redux
- [redux-thunk](https://www.npmjs.com/package/redux-thunk): Middleware that allows you to write the action creators in a way that results in more compartmentalized code. More on this later.
- [redux-devtools-extension](https://www.npmjs.com/package/redux-devtools-extension): Makes the Redux state container visible to the browsers Dev Tools.

## Setting up files

_./src/index.js_

```jsx
import React from "react"
import ReactDOM from "react-dom"
import { Provider } from "react-redux"
import App from "./App"
import store from "./store"

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
)
```

So here `<App />` is our main component that contains every other component you create. `<Provider />` is a wrapper component that makes the Redux state container available to every component that is wrapped in the `connect()` function. I will show how that is done later.

_./src/store.js_

```jsx
import { createStore, applyMiddleware, combineReducers } from "redux"
import thunk from "redux-thunk"
import { composeWithDevTools } from "redux-devtools-extension"
import reducer1 from "./reducers/reducer1"
import reducer2 from "./reducers/reducer2"

const reducer = combineReducers({
  state1: reducer1,
  state2: reducer2,
})

const store = createStore(reducer, composeWithDevTools(applyMiddleware(thunk)))

export default store
```

This is where we setup the state container of the application. I already use two example reducers, because that way the `combineReducers` function is at place. Later when you need to add new reducers, you just need to list them here.

_./src/App.js_

```jsx
import React from "react"
import { connect } from "react-redux"
import Subcomponent from "./components/Subcomponent"
import { addThis } from "./reducers/reducer1"
import { addThat } from "./reducers/reducer2"

const App = props => {
  return (
    <div>
      <h1>App contains its subcomponent</h1>
      <Subcomponent />
    </div>
  )
}

const mapStateToProps = state => {
  return {
    state1: state.state1,
    state2: state.state2,
  }
}

const mapDispatchToProps = {
  addThis,
  addThat,
}

export default connect(mapStateToProps, mapDispatchToProps)(App)
```

Here we import the `connect()` function, which gives us access to the Redux store. We also import the `Subcomponent` and action creators from the reducer files.

In `mapStateToProps` function we define which state the component in question needs access to. For the sake of this example, we give it access to both state, but it could be only one as well. These states are accessible from the App via `props`.

In `mapDispatchToProps` we list all of our action creator function. They need to be function that we will define next in the reducer files. These functions are accessible from the App component by calling e.g `props.addThis(content)`.

Lastly, we connect our app the the Redux store by calling `connect(mapStateToProps, mapDispatchToProps)(App)`.

_./src/components/Subcomponent.js_

```jsx
import React from "react"
import { connect } from "react-redux"
import { addThis } from "../reducers/reducer1"
import { addThat } from "../reducers/reducer2"

const Subcomponent = props => {
  return <h2>Subcomponent</h2>
}

const mapStateToProps = state => {
  return {
    state1: state.state1,
    state2: state.state2,
  }
}

const mapDispatchToProps = {
  addThis,
  addThat,
}

export default connect(mapStateToProps, mapDispatchToProps)(Subcomponent)
```

Nothing special here. At this point the component is basically identical to the App component.

_./src/reducers/reducer1.js_

```jsx
const reducer = (state = "", action) => {
  switch (action.type) {
    case "DO_THIS":
      return action.data
    default:
      return state
  }
}

export const addThis = content => {
  return dispatch => {
    dispatch({
      type: "DO_THIS",
      data: content,
    })
  }
}

export default reducer
```

_./src/reducers/reducer2.js_

```jsx
const reducer = (state = "", action) => {
  switch (action.type) {
    case "DO_THAT":
      return action.data
    default:
      return state
  }
}

export const addThat = content => {
  return dispatch => {
    dispatch({
      type: "DO_THAT",
      data: content,
    })
  }
}

export default reducer
```

The reducers are equal in this example. The action creator functions are to be called from the components. For example, in order to modify `state1` from the App component you would call `props.addThis(new state)` and the action creator function in the _reducer1_ file would dispatch the action as a plain javascript object to the reducer, which would return the new state.

Because we used also the `redux-thunk` middleware, we could also you async function in the action creator function. We could for example, do something like this:

```jsx
export const addThat = content => {
  return async dispatch => {
    const data = await axios.get(url)
    dispatch({
      type: "DO_THAT",
      data,
    })
  }
}
```

Doing an API call in the action creator function can be extremely useful and it will simplify the logic in the React components.

## Summary

I wrote this blog post in order to clarify what is actually needed to start a React with Redux project. Sometimes it can get confusing with all the different libraries and documentation pages open in the browser. When you want to get started fast it's good to have a standard boilerplate ready to go and use it as a starting point.
