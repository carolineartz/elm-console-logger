# Logger for Redux

[![npm](https://img.shields.io/npm/v/redux-logger.svg?maxAge=2592000?style=plastic)](https://www.npmjs.com/package/redux-logger)
[![npm](https://img.shields.io/npm/dm/redux-logger.svg?maxAge=2592000?style=plastic)](https://www.npmjs.com/package/redux-logger)
[![Build Status](https://travis-ci.org/evgenyrodionov/redux-logger.svg?branch=master)](https://travis-ci.org/evgenyrodionov/redux-logger)

![redux-logger](http://i.imgur.com/CgAuHlE.png)

Thank you to our sponsor [LogRocket](https://logrocket.com?cid=github_redux)!

[![](https://i.imgur.com/Yp5mUx2.png)](https://logrocket.com?cid=github_redux)

> LogRocket is a production Redux logging tool that lets you replay problems as if they happened in your own browser. Instead of guessing why errors happen, or asking users for screenshots and log dumps, LogRocket lets you replay Redux actions + state, network requests, console logs, and see a video of what the user saw.

## Table of contents

- [Install](#install)
- [Usage](#usage)
- [Options](#options)
- [Recipes](#recipes)
  - [Log only in development](#log-only-in-development)
  - [Log everything except actions with certain type](#log-everything-except-actions-with-certain-type)
  - [Collapse actions with certain type](#collapse-actions-with-certain-type)
  - [Transform Immutable (without `combineReducers`)](#transform-immutable-without-combinereducers)
  - [Transform Immutable (with `combineReducers`)](#transform-immutable-with-combinereducers)
  - [Log batched actions](#log-batched-actions)
- [To Do](#to-do)
- [Known issues](#known-issues) (with `react-native` only at this moment)
- [License](#license)

## Install

`npm i --save redux-logger`

## Usage

```javascript
import { applyMiddleware, createStore } from "redux";

// Logger with default options
import logger from "redux-logger";
const store = createStore(reducer, applyMiddleware(logger));

// Note passing middleware as the third argument requires redux@>=3.1.0
```

Or you can create your own logger with custom [options](https://github.com/evgenyrodionov/redux-logger#options):

```javascript
import { applyMiddleware, createStore } from "redux";
import { createLogger } from "redux-logger";

const logger = createLogger({
  // ...options
});

const store = createStore(reducer, applyMiddleware(logger));
```

Note: logger **must be** the last middleware in chain, otherwise it will log thunk and promise, not actual actions ([#20](https://github.com/evgenyrodionov/redux-logger/issues/20)).

## Options

```javascript
{
  predicate, // if specified this function will be called before each action is processed with this middleware.
  collapsed, // takes a Boolean or optionally a Function that receives `getState` function for accessing current store state and `action` object as parameters. Returns `true` if the log group should be collapsed, `false` otherwise.
  duration = false: Boolean, // print the duration of each action?
  timestamp = true: Boolean, // print the timestamp with each action?

  level = 'log': 'log' | 'console' | 'warn' | 'error' | 'info', // console's level
  colors: ColorsObject, // colors for title, prev state, action and next state: https://github.com/evgenyrodionov/redux-logger/blob/master/src/defaults.js#L12-L18
  titleFormatter, // Format the title used when logging actions.

  stateTransformer, // Transform state before print. Eg. convert Immutable object to plain JSON.
  actionTransformer, // Transform action before print. Eg. convert Immutable object to plain JSON.
  errorTransformer, // Transform error before print. Eg. convert Immutable object to plain JSON.

  logger = console: LoggerObject, // implementation of the `console` API.
  logErrors = true: Boolean, // should the logger catch, log, and re-throw errors?

  diff = false: Boolean, // (alpha) show diff between states?
  diffPredicate // (alpha) filter function for showing states diff, similar to `predicate`
}
```

### Options description

#### **level (String | Function | Object)**

Level of `console`. `warn`, `error`, `info` or [else](https://developer.mozilla.org/en/docs/Web/API/console).

It can be a function `(action: Object) => level: String`.

It can be an object with level string for: `prevState`, `action`, `nextState`, `error`

It can be an object with getter functions: `prevState`, `action`, `nextState`, `error`. Useful if you want to print
message based on specific state or action. Set any of them to `false` if you want to hide it.

- `prevState(prevState: Object) => level: String`
- `action(action: Object) => level: String`
- `nextState(nextState: Object) => level: String`
- `error(error: Any, prevState: Object) => level: String`

_Default: `log`_

#### **duration (Boolean)**

Print duration of each action?

_Default: `false`_

#### **timestamp (Boolean)**

Print timestamp with each action?

_Default: `true`_

#### **colors (Object)**

Object with color getter functions: `title`, `prevState`, `action`, `nextState`, `error`. Useful if you want to paint
message based on specific state or action. Set any of them to `false` if you want to show plain message without colors.

- `title(action: Object) => color: String`
- `prevState(prevState: Object) => color: String`
- `action(action: Object) => color: String`
- `nextState(nextState: Object) => color: String`
- `error(error: Any, prevState: Object) => color: String`

#### **logger (Object)**

Implementation of the `console` API. Useful if you are using a custom, wrapped version of `console`.

_Default: `console`_

#### **logErrors (Boolean)**

Should the logger catch, log, and re-throw errors? This makes it clear which action triggered the error but makes "break
on error" in dev tools harder to use, as it breaks on re-throw rather than the original throw location.

_Default: `true`_

#### **collapsed = (getState: Function, action: Object, logEntry: Object) => Boolean**

Takes a boolean or optionally a function that receives `getState` function for accessing current store state and `action` object as parameters. Returns `true` if the log group should be collapsed, `false` otherwise.

_Default: `false`_

#### **predicate = (getState: Function, action: Object) => Boolean**

If specified this function will be called before each action is processed with this middleware.
Receives `getState` function for accessing current store state and `action` object as parameters. Returns `true` if action should be logged, `false` otherwise.

_Default: `null` (always log)_

#### **stateTransformer = (state: Object) => state**

Transform state before print. Eg. convert Immutable object to plain JSON.

_Default: identity function_

#### **actionTransformer = (action: Object) => action**

Transform action before print. Eg. convert Immutable object to plain JSON.

_Default: identity function_

#### **errorTransformer = (error: Any) => error**

Transform error before print.

_Default: identity function_

#### **titleFormatter = (action: Object, time: String?, took: Number?) => title**

Format the title used for each action.

_Default: prints something like `action @ ${time} ${action.type} (in ${took.toFixed(2)} ms)`_

#### **diff (Boolean)**

Show states diff.

_Default: `false`_

#### **diffPredicate = (getState: Function, action: Object) => Boolean**

Filter states diff for certain cases.

_Default: `undefined`_

## Recipes

### Log only in development

```javascript
const middlewares = [];

if (process.env.NODE_ENV === `development`) {
  const { logger } = require(`redux-logger`);

  middlewares.push(logger);
}

const store = compose(applyMiddleware(...middlewares))(createStore)(reducer);
```

### Log everything except actions with certain type

```javascript
createLogger({
  predicate: (getState, action) => action.type !== AUTH_REMOVE_TOKEN
});
```

### Collapse actions with certain type

```javascript
createLogger({
  collapsed: (getState, action) => action.type === FORM_CHANGE
});
```

### Collapse actions that don't have errors

```javascript
createLogger({
  collapsed: (getState, action, logEntry) => !logEntry.error
});
```

### Transform Immutable (without `combineReducers`)

```javascript
import { Iterable } from "immutable";

const stateTransformer = state => {
  if (Iterable.isIterable(state)) return state.toJS();
  else return state;
};

const logger = createLogger({
  stateTransformer
});
```

### Transform Immutable (with `combineReducers`)

```javascript
const logger = createLogger({
  stateTransformer: state => {
    let newState = {};

    for (var i of Object.keys(state)) {
      if (Immutable.Iterable.isIterable(state[i])) {
        newState[i] = state[i].toJS();
      } else {
        newState[i] = state[i];
      }
    }

    return newState;
  }
});
```

### Log batched actions

Thanks to [@smashercosmo](https://github.com/smashercosmo)

```javascript
import { createLogger } from "redux-logger";

const actionTransformer = action => {
  if (action.type === "BATCHING_REDUCER.BATCH") {
    action.payload.type = action.payload.map(next => next.type).join(" => ");
    return action.payload;
  }

  return action;
};

const level = "info";

const logger = {};

for (const method in console) {
  if (typeof console[method] === "function") {
    logger[method] = console[method].bind(console);
  }
}

logger[level] = function levelFn(...args) {
  const lastArg = args.pop();

  if (Array.isArray(lastArg)) {
    return lastArg.forEach(item => {
      console[level].apply(console, [...args, item]);
    });
  }

  console[level].apply(console, arguments);
};

export default createLogger({
  level,
  actionTransformer,
  logger
});
```

## To Do

- [x] Update eslint config to [airbnb's](https://www.npmjs.com/package/eslint-config-airbnb)
- [ ] Clean up code, because it's very messy, to be honest
- [ ] Write tests
- [ ] Node.js support
- [ ] React-native support

Feel free to create PR for any of those tasks!

## Known issues

- Performance issues in react-native ([#32](https://github.com/evgenyrodionov/redux-logger/issues/32))

## License

MIT
