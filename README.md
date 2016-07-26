> ### Version 2.0 coming soon
> Almost immediately after releasing 1.0 I've noticed some limitations. 2.0 will have more flexibility and power, as well as a simpler API. Win-win!

# horizon-redux
A small library that helps you connect Horizon.io with Redux in a flexible, non-intrusive way.

[![Build Status](https://travis-ci.org/shanecav/horizon-redux.svg?branch=master)](https://travis-ci.org/shanecav/horizon-redux)
[![Coverage Status](https://coveralls.io/repos/github/shanecav/horizon-redux/badge.svg?branch=master)](https://coveralls.io/github/shanecav/horizon-redux?branch=master)

## What does it do?
horizon-redux provides a light-weight (<1kb minified + gzipped) and flexible interface for connecting [Redux](https://github.com/reactjs/redux) with [Horizon.io](http://horizon.io/). It accomplishes this by providing two functions: `createMiddleware` and `setupSubscriptionActions`.

`createMiddleware` is used to create a Redux middleware that watches for specific actions, and makes corresponding Horizon queries when those actions are dispatched.

`setupSubscriptionActions` is used to create Horizon query subscriptions that respond by dispatching corresponding actions every time the subscription receives new data. Subscriptions can be a one-time `fetch()`, or a continuous `watch()`.

This approach allows you to use Redux to manage your app's entire state, as opposed to having external Horizon.io bindings directly to your UI components. This way, you can enjoy the simplicity of Horizon.io without losing the benefits of a well-structured Redux app.

horizon-redux has zero npm dependencies, and its only requirements are Horizon.io and Redux.

## Installation

`npm i -S horizon-redux`

## Usage

```js
import { compose, applyMiddleware, createStore } from 'redux'
import Horizon from '@horizon/client'
import HorizonRedux from 'horizon-redux'

import { ADD_MESSAGE, addMessageSuccess, addMessageFailure, newMessages } from '../actions/chat'

const hz = Horizon({ host: 'localhost:8181' })

// Create Redux middleware that watches for specific actions and runs a corresponding Horizon query
const hzMiddleware = HorizonRedux.createMiddleware(
  hz,
  {
    [ADD_MESSAGE]: (horizon, action, dispatch) => horizon('messages').store(action.payload)
  }
)

// Create the Redux store with horizon-redux middleware
const store = createStore(
  rootReducer,
  window.initialState,
  applyMiddleware(hzMiddleware)
)

// Set up Redux action(s) to dispatch whenever a corresponding Horizon client subscription receives new data
HorizonRedux.setupSubscriptionActions(
  hz,
  store.dispatch,
  [{
    query: (horizon) => horizon('messages').order('datetime', 'descending').limit(10).watch(),
    actionCreator: newMessages
  }]
)
```

## API

### createMiddleware(horizonInstance, config)

Creates a Redux middleware that watches for specific actions, and makes corresponding Horizon queries when those actions are dispatched. (The action is always pushed through the middleware chain, so other middlewares/reducers can respond to it as well if needed.)

#### Arguments:

1. `horizonInstance` - A Horizon.io client instance.
2. `config` - An object where each key is an action type, and each corresponding value is a function that takes a horizon instance, action, and Redux store dispatch method as arguments and runs a Horizon Collection query.

#### Returns:

Redux middleware

#### Example:
```js
const hzMiddleware = HorizonRedux.createMiddleware(horizonInstance, {
  [ADD_MESSAGE_REQUEST]: (horizon, action, dispatch) => {
    horizon('messages').store(action.payload).subscribe(
      (id) => dispatch(addMessageSuccess(id, action.payload)),
      (err) => dispatch(addMessageFailure(err, action.payload))
    )
  }
  // etc...
})
```

---

### setupSubscriptionActions(horizonInstance, dispatch, config)

Creates Horizon query subscriptions that respond by dispatching corresponding actions every time the subscription receives new data. Subscriptions can be a one-time `fetch()`, or a continuous `watch()`.

#### Arguments:

1. `horizonInstance` - A Horizon.io client instance
2. `dispatch` - Redux store dispatch method
3. `config` - An array of objects with the following properties:
  * `query` - Function that takes a Horizon instance and returns a Horizon query without .subscribe() method.
  * `actionCreator` - Function that takes Horizon query result data and returns an action using that data.
  * `onQueryError` - (optional) Function that takes a Horizon query error and handles it. Defaults to `(err) => throw new Error(err)`.

#### Returns:

n/a

#### Example:

```js
HorizonRedux.setupSubscriptionActions(horizonInstance, store.dispatch, [
  {
    query: (horizon) => horizon('messages').order('datetime', 'descending').limit(10).watch(),
    actionCreator: refreshMessages,
    onQueryError: (err) => {
      console.error('error receiving new messages from horizon server')
    }
  }
  // etc...
])
```

## License

[MIT](LICENSE)
