
# Learning Redux

---

## About Eric

* @ericmasiello
* eric.j.masiello@gmail.com
* Fullstack JavaScript Engineer @ Vistaprint Digital
* Co-Instructor for FEWD 31
* Co-Author of Mastering React Native


---

## Agenda

1. Review: ES6, Promises, & React
2. Managing state in React w/o Redux
3. Redux: Actions, Reducers
4. Middleware as Dev Tools
5. Presentational vs. Container Components
6. Middleware for Async Actions with `redux-thunk`
7. Selectors with `reselect`

---

## Review: ES6, Promises, & React

--

### Arrow Functions (aka fat arrows)

* Always anonymous
* Bind `this` context to parent's `this`

```js
function (foo, bar) {
  return foo + bar;
}
// or
(foo, bar) => {
  return foo + bar;
}

// or
(foo, bar) => foo + bar;
```

--

### Destructuring
* Can be used on objects or arrays
* Easy way to peal off props from an object
* Easy to to extract values from an array 

```js
const list = ['a', 'b', 'c'];
const person = {
  firstName: 'Eric',
  lastName: 'Masiello',
  employer: 'Vistaprint Digital',
};

const [first, second] = list;
// first === 'a'
// second === 'b'

const { firstName, lastName } = person;
// firstName === 'Eric'
// lastName === 'Masiello'
```

--
### Rest Parameter
* Can be used with arrays and function arguments
* Applied using `...` before the variable

```js
const list = ['a', 'b', 'c'];

const [first, ...everythingElse] = list;
// first === 'a'
// everythingElse contains ['b', 'c'];

function foo(first, second, ...theRest) {
  // first === 'c'
  // second === 'c'
  // theRest contains ['c', 'd', 'e']
}
foo('a', 'b', 'c', 'd', 'e');
```

--
### Spread Operator

* Takes the parts of an array and *spreads* them out
* Also uses the `...` syntax

```js
const parts = ['shoulders', 'knees']; 
const lyrics = ['head', ...parts, 'and', 'toes']; 
// ["head", "shoulders", "knees", "and", "toes"]
```
* Can be used to copy the contents of an array

```js
const a = ['a', 'b', 'c'];
const copyOfA = [...a];
// a !== copyOfA (copies contents of array, not a pointer!)
copyOfA.push('d');
// a contains ['a', 'b', 'c']
// copyOfA contains ['a', 'b', 'c', 'd']
```

--

### Rest with Objects
* **Disclaimer:** Currently requires Babel for support
* Useful in React when dealing with `props`

```js
const person = {
  firstName: 'Eric',
  lastName: 'Masiello',
  employer: 'Vistaprint Digital',
};

// Using Rest
const { employer, ...nameProps } = person;
// employer === 'Vistprint DigitalEric'
// nameProps contains { firstName: 'Eric', lastName: 'Masiello' }
```

--

### Using Rest with Objects in Function Parameters
* **Disclaimer:** Currently requires Babel for support
* Similar to object destructuring, the *keys* must match up

```js
function foo({ first, second, ...theRest }) {
  // first === 'a',
  // second === 'b',
  // theRest contains { third: 'c', fourth: 'd' }
}
foo({
  first: 'a',
  second: 'b',
  third: 'c',
  fourth: 'd',
});
```

--

### Using Spread with Objects
* **Disclaimer:** Currently requires Babel for support
* Great for passing down `props` to React elements

```js
const headerProps = {
  firstName: 'Eric',
  lastName: 'Masiello',
  className: 'vip',
};

return (
  <div>
    <Header {...headerProps} />
  </div>
);
```

--

### Default function arguments
Can set default value when argument is `undefined`

```js
function doStuff(list = [], filterBy = 'stuff') {

}

doStuff(); // list =[], filterBy = 'stuff'
doStuff(['a', 'b']); // list = ['a', 'b'], filterBy = 'stuff'
doStuff(undefined, 'things'); // list = [], filterBy = 'things'
doStuff(null, null); // list = null, filterBy = null
```
Only works when value is `undefined` (not `null`)

--

### Promises
* Similar to callbacks
* Unlike callbacks, no "inversion of control"
* Executed asynchronously when calling `.then()`

```js
const p = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve('It works!');
  }, 2000);
});
```

--

### More Promises
* You *get* the result by calling `.then()`
* Promises can be chained
* Promises are also called "thenables"

```js
p.then((result) => {
  // result === 'It works!';
  return result + ' Hurray!';
}).then((result) => {
  console.log(result); // 'It works! Hurray!';
});
```

--

### Handling Errors with Promises
Promises also have a `.catch()` method for handling erros in the promise chain

```js
const p = new Promise(function(resolve, reject) {
  setTimeout(function() {
    const error = new Error('Ah dang!');
    reject(error);
  }, 2000);
});

p.then((result) => {
  console.log('it wont get here');
}).catch((error) => {
  console.error(error.message); // 'Ah dang!'
});
```

--

### React Components

[CodePen Example](https://codepen.io/ericmasiello/pen/mmEwbq?editors=0010)

```js
// Class (stateful) component
class Greeting extends React.Component {
  constructor(props) {
    super(props);
    this.state = { greeting: 'Hello' };
    this.toggleGreeting = this.toggleGreeting.bind(this);
  }
  toggleGreeting() {
    this.setState({
      greeting: this.state.greeting === 'Hello' ? 'Bye' : 'Hello',
    });
  }
  render() {
    return (
      <button onClick={this.toggleGreeting}>
        {this.state.greeting}, {this.props.firstName}!
      </button>
    );
  }
}

// Functional (stateless) Component
function Header(props) {
  return (
    <div className="header">
      <Greeting firstName="Eric" />
    </div>
  );
}
```


---

## Managing state in React w/o Redux

--

### React `state`

* `state` is used as a data store for the app
```js
  constructor(props) {
    super(props);
    this.state = {
      news: [],
      searchTerm: '',
    };
  }
```
* It is often spread across multiple components, each with their own `state`

--

### Upsides to `state`

* Low barrier to entry
* Doesn't require any additional libraries

--

### Downsides to `state`

* Can't be used to server-side render React components
* No single source of truth (multiple component `state`s)
* Requires `React.createClass` or extending `React.Component`

Note:
* At scale, can be difficult to reason about

--

### Alternatives to `state`

* MobX https://github.com/mobxjs/mobx
* Redux http://redux.js.org/
* Reflux https://github.com/reflux/refluxjs
* Or any other one of the million flux libraries

--

### Why Redux

* Huge ecosystem of middleware
* Great dev tooling (e.g. time travel debugging)
* Great documentation http://redux.js.org/
* De facto choice for React state management
* Well supported by other libraries/frameworks
* Supports React server side rendering

---

![Redux](img/redux.png)

## Actions, Reducers, <del>and Middleware</del>

(We'll talk about middleware later on)

--

### A single source of truth

* The whole state of your app is stored in an object tree inside a single store.

```js
  {
    isLoading: false,
    news: [{}, {}, {}],
    searchTerm: 'general assembly',
    bookmarks: [733, 522, 28],
    userProfile: {
      firstName: 'Eric',
      lastName: 'Masiello',
      email: 'eric.j.masiello@gmail.com',
      jwt: 'xxxxx.yyyyy.zzzzz',
    },
  }
``` 


\*"Ephemeral data" can still be stored in `state`

--

### Synchronous

* Redux only handles synchronous data flow
* Must apply middleware to work asynchronously (e.g. `redux-thunk`, `redux-promise`, `redux-saga`)

--

### Updating State with Actions

* Redux state is read only
* Changes are made via dispatched **actions**
* Actions have a `type` property

```js
store.dispatch({ type: 'IS_LOADING' });

store.dispatch({ type: 'DONE_LOADING' })

store.dispatch({ type: 'UPDATE_SEARCH_TERM', payload: 'gener' })

```
*Think of actions as the "breadcrumbs" of what's changed*

--

### Action Creators

* A factory that returns an action
```js
  // searchTermActions.js
  function updateSearchTerm(searchTerm) {
    return {
      type: 'UPDATE_SEARCH_TERM',
      payload: searchTerm,
    };
  }

  // SearchComponent.js
  store.dispatch(updateSearchTerm('gener'));
```
* Can be synchronous or asynchronous
* Asynchronous requires use of middleware to dispatch other actions

--

### Reducers

* Functions that decide how each action transforms the their respective piece of state
* Each reducer maps to exactly 1 part of your state tree object

```js
  {
    isLoading: loadingReducer,
    news: newsReducer,
    searchTerm: searchTermReducer,
    bookmarks: bookMarksReducer,
    userProfile: userProfileReducer,
  }
``` 
* Receive two arguments: the *current state* and the *dispatched action*
* Must be *pure functions*
* Must be side-effect free

Note:
- Pure functions are ones that do not rely on outside state. For any given input, you'll always get the exact same output
- Side-effects are things that affect state outside of the function, e.g. updating or reading from the DOM, ajax requests to the server, technically even console.logs but we can forgive that one

--

### Reducers

* Reducers **return new state** for their respective piece of the state tree object

```js
function loadingReducer(state = false, action) {
  switch (action.type) {
  case 'IS_LOADING':
    return true;
  case 'DONE_LOADING':
    return false;
  default:
    return state;
  }
}

function searchTermReducer(state = '', action) {
  switch (action.type) {
  case 'UPDATE_SEARCH_TERM':
    return action.payload;
  default:
    return state;
  }
}
```
* **Note**: We are not changing state directly

--

### Why is it called a "reducer?"

```js
[1, 2, 3, 4].reduce((acc, value) => acc + value, 0);
```

"In Redux, the accumulated value is the state object, and the values being accumulated are actions. Reducers calculate a new state given the previous state and an action."

--

### Creating a Redux store

```js
import { createStore, combineReducers, applyMiddleware } from 'redux';

const store = createStore(
  combineReducers({
    isLoading: loadingReducer,
    news: newsReducer,
    searchTerm: searchTermReducer,
    bookmarks: bookMarksReducer,
    userProfile: userProfileReducer,
  }),
  STATE_FROM_SERVER, // optional
  applyMiddleware(logger, thunk), // optional
);
```

--

## Listening for store updates

* `subscribe` method from store allows you to listen for store updates

```js
  constructor(props) {
    super(props);
    this.state = {
      isLoading: false,
      news: [],
      searchTerm: '',
      bookmarks: [],
      userProfile: {},
    };

    this.onStoreUpdate = this.onStoreUpdate.bind(this);
    store.subscribe(this.onStoreUpdate);
  }

  onStoreUpdate() {
    this.setState(store.getState());
  }
```

--

## Redux Data Flow

<img src="img/redux-data-flow.png" style="max-height: 450px; border-width: 0; box-shadow: none;" />

*Note:* This does not account for middleware

--

## Code Along
### Synchronous Data Flow
1. Add `redux`
2. Create a `store` with a working `newsReducer`
3. Create a `LOAD_NEWS` action and action creator (use `data.json`)
4. Dispatch the `LOAD_NEWS` action in place of `nytFetch('technology')`

---

## Middlware as Dev Tools

--

## What is Redux Middleware

* Similar to Express middleware
* Sits between your action creators and reducers
* Can block or dispatch additional actions
* Has full visibility into action and the state of your app
* Can also be used to handle async actions (more on this later)
* Are applied to the store using `applyMiddleware`

--

## Redux Logger

Logs to your console...

1. The state before the action
2. The dispatched action
3. The state after the action has passed through the reducers

--

## Code Along
### Add Redux Logger
1. Install `redux-logger`
2. Add it as middleware when calling `createStore` using `applyMiddleware`

--

## Redux DevTools

https://github.com/zalmoxisus/redux-devtools-extension

1. Exposes all actions and a timeline
2. Allows for time travel debugging
3. Allows you to dispatch actions from the tool
4. Available as a Chrome Extension, Firefox Extension, or Electron app

--

## Word of Caution

*You'll likely want to strip out Redux DevTools and Redux Logger in production*

--

## Code Along
### Configure Redux DevTools

1. Install the Chrome Extension https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd
2. Add the Redux DevTools as the composer

---

## Problem: We've added Redux, but we're still using `state`.

---

## Presentational vs. Container Components
### Using `react-redux`

--

### Presentational Components

* Concerned with how things *look*
* Do not specify how the data is mutated or loaded
* Are passed data and callbacks via `props`
* Rarely use `state`
* Typically written as stateless/functional component

```js
function Button(props) {
  return (
    <button className={props.className} onClick={props.onClickHandler}>
      {props.children}
    </button>
  );
}
```

--

### Container Components
* Concerned with things *work*
* Typically don't contain any presentation (maybe just a `div`)
* Pass down data and callbacks to other components as `props`
* Callbacks include *action creators*

--

### Primary Benefits of this Approach
1. Separation of concerns: UI (Presentation) vs. Data
2. Easier to make reusable UI components

--

### `react-redux` Container components

* Official bindings for creating container components using React and Redux
* Creates container using `connect` higher order function
* Autowraps action creators with `dispatch`

```js
function mapStateToProps(state) {
  return {
    news: state.news,
  };
}

const NewsArchiveContainer = connect(mapStateToProps, { loadNews })(NewsArchive);

```
--
### `react-redux` Provider

* "Connected" Container components are exposed the `store` via `Provider`
* Typically `Provider` wraps the top level of your app
* Exposes `store` via React's `context`

```html
<Provider store={store}>
  <Router>
    <div className="App">
      <AppHeader />
      <div className="App-main">
        <Route exact path="/" component={NewsArchiveContainer}/>
        <Route path="/bookmarks" component={Bookmarks}/>
        <Route path="/profile" component={Profile}/>
      </div>
    </div>
  </Router>
</Provider>
```
--

### Architecture
* App has many presentational components
* App contains a select number of container components that compose the different views
* Container components wrap many presentational components

```html
<Provider store={store}>
  <App>
    <NewsContainer>
      <News />
      <SideBar />
    </NewsContainer>
    <UserProfileContainer>
      <UserProfile />
    </UserProfileContainer>
  </App>
</Provider>
```

* Note: Container components do not need to be at the top of your tree.

--

## Code Along (Pt. 1)
### Refactor `NewsArchive` to use container
1. Create `NewsArchiveContainer` that passes down `news` and `loadNews`
2. Refactor `NewsArchive` to call `loadNews` on `componentDidMount`
3. Refactor `App` to a functional component that loads `NewsArchiveContainer` and wraps itself in `Provider`

--

## Code Along (Pt. 2)
### Refactor `Search` to use a container
1. Create `SearchContainer` that passes down `searchTerm` as `value` and custom binds the action creator `searchNews` to `onSearch`
2. Create `SEARCH_NEWS` action, appropriate action creator, and reducers
3. Update `App` to use the `SearchContainer` component

---

## Middleware for Async Actions with `redux-thunk`

--

### The Basic Middleware Flow

Redux out of the box only works synchronously

1. Action creators return actions
2. Actions flow through middleware
3. Actions pass on to reducers
4. Reducers return update state to the store

--

### Action Creators + Middlware sitting in a tree

* Action creators can return other data types (e.g. functions, promises, etc.)
* Middleware can inspect these "actions", stop them from passing to the reducers, and operate on them.

--

### Example: Dispatching a Promise
#### Using `redux-promise`

```js
function loadNews() {
  return {
    type: LOAD_NEWS,
    payload: nytFetch('technology'), // <-- Ooo! A Promise!
  };
}
```

--

### Example: Dispatching a Promise

<a href="img/redux-promise.png" target="_new"><img src="img/redux-promise.png" style="max-height: 700px; border-width: 0; box-shadow: none;" /></a>

--

### Example: Dispatching a Function
#### Using `redux-thunk`

```js
function loadNews() {
  return function(dispatch) {
    // dispatch something to let our UI know its fetching data
    dispatch({
      type: IS_LOADING,
    });

    nytFetch('technology')
    .then(news => {
      // When the data returns, dispatch
      // another action with the data in tact
      dispatch({
        type: LOAD_NEWS,
        payload: news,
      });
    });
  }
}
```

--

### More on Redux Thunk
* Action creators return a function instead of an action object
* Action creators are "thunks"
* The returned function is executed by Redux Thunk middleware and passes the `dispach` function to the function

--

### What's a thunk?

A thunk is a function that wraps an expression to delay its evaluation.

```js
// calculation of 1 + 2 is immediate
// x === 3
let x = 1 + 2;
```
```js
// calculation of 1 + 2 is delayed
// foo can be called later to perform the calculation
// foo is a thunk!
let foo = () => 1 + 2;
```

Note:
- Code example from https://github.com/gaearon/redux-thunk

--

### Code Along
#### Add Redux Thunk

1. Install and apply `redux-thunk` middleware
2. Remove `data.json` and load the data using `nytFetch` and a thunk
3. add `isLoading` to the state tree along w/ a reducer
4. Update `NewsArchive` to display "Loading..." when appropriate

---

## Selectors with `reselect`

--

### Reselect Selectors

* Compute derived data from Redux state
* Combined data is then passed down to React components
* Memoizes inputs/outputs

--

### Types of Selectors
#### Basic Input Selectors
* Functions that return data from Redux state
* Must not transform data

#### Memoized Selectors
* Created using `createSelector`
* Take basic selectors as inputs
* Return combined state to components

--

### Selectors

```js
const newsSelector = state => state.news;
const searchTermSelector = state => state.searchTerm;

const caseInsensitiveSearchTermSelector = createSelector(
  [searchTermSelector],
  searchTerm => searchTerm.toLowerCase()
);

export const searchNewsSelector = createSelector(
  [newsSelector, caseInsensitiveSearchTermSelector],
  (newsItems, searchTerm) => {
    // ...
    return filteredNews;
  },
);
```

--

## Code Along
1. Install `reselect`
2. Create news selectors that make it possible to search the news
3. Pass filtered `news` to `NewsArchive` as props

---

## Thank you!

