
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
2. Managing state w/o Redux
3. Redux: Actions, Reducers, and Middleware
4. Presentational vs. Container Components
5. Refactoring with `react-redux`
6. Async Redux with `redux-thunk`
7. Selectors with `reselect`

---

## Review: ES6, Promises, & React

---

## Managing state w/o Redux

--

### React `state`

* `state` is often used as a data store for data
```js
  constructor(props) {
    super(props);
    this.state = {
      news: [],
      searchTerm: '',
    };
  }
```
* Often spread across multiple components, each with their own `state`

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
* Great dev tooling (time travel debugging)
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

---

## Problem: We've added Redux, but we're still using `state`.

---

## Presentational vs. Container Components

--

### Presentational Components

* Concerned with how things *look*
* Do not specify how the data is mutated or loaded
* Are passed data and callbacks via `props`
* Rarely use `state`
* Typically written as functional component

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
* In Redux, created used `connect`

```js
function mapStateToProps(state) {
  // ...
}

const NewsArchiveContainer = connect(mapStateToProps, { loadNews })(NewsArchive);

```

--

### Architecture
* App has many stateless presentational components
* App contains a select number of container components that compose the different views
* Container components wrap many presentational components

```html
<App>
  <NewsContainer>
    <News />
    <SideBar />
  </NewsContainer>
  <UserProfileContainer>
    <UserProfile />
  </UserProfileContainer>
</App>
```

---

## Refactoring with `react-redux`

---

## Async Redux with `redux-thunk`

---

## Selectors with `reselect`

---

## Thank you!

