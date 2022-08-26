# useReducer: simple Counter

## Background

React's `useState` hook can get you a really long way with React state management. That said, sometimes you want to separate the state logic from the components that make the state changes. In addition, if you have multiple elements of state changes. In addition, if you have multiple elements of state that typically change together, then having an object that contains those elements of state can be quite helpful.

This is where `useReducer` comes in really handy.

Typically, you'll use `useReducer` with an object of state, but we're going to start by managing a single number (a `count`).

Here's an example of using `useReducer` to manage the value of a name in an input.

```JSX
function nameReducer(previousName, newName) {
  return newName
}

const initialNameValue = 'Joe'

function NameInput() {
  const [name, setName] = React.useReducer(nameReducer, initialNameValue)
  const handleChange = event => setName(event.target.value)
  return (
    <>
      <label>
        Name: <input defaultValue={name} onChange={handleChange} />
      </label>
      <div>You typed: {name}</div>
    </>
  )
}
```

One important thing to note here is that the reducer (called `nameReducer` above) is called with two arguments:

> 1. the current state
> 2. whatever it is that the dispatch function (called `setName` above) is called with. This is often called an 'action.'

`https://kentcdodds.com/blog/should-i-usestate-or-usereducer`
`https://kentcdodds.com/blog/how-to-implement-usestate-with-usereducer`

## Other notes

### lazy initialization

Sometimes lazy initialization can be useful.

```JSX
function init(initialStateFromProps) {
  return {
    pokemon: null,
    loading: false,
    error: null,
  }
}

// ...

const [state, dispatch] = React.useReducer(reducer, props.initialState, init)
```

So, if you pass a third function argument to `useReducer`, it passes the second argument to that function and uses the return value for the initial state.

This could be useful if our `init` function read into localStorage or something else that we wouldn't want happending every re-render.

### The full `useReducer` API

If you're into TypeScript, here's some type definitions for `useReducer`:

> `https://levelup.gitconnected.com/usetypescript-a-complete-guide-to-react-hooks-and-typescript-db1858d1fb9c`

```JSX
type Dispatch<A> = (value: A) => void
type Reducer<S, A> = (prevState: S, action: A) => S
type ReducerState<R extends Reducer<any, any>> = R extends Reducer<infer S, any>
  ? S
  : never
type ReducerAction<R extends Reducer<any, any>> = R extends Reducer<
  any,
  infer A
>
  ? A
  : never

function useReducer<R extends Reducer<any, any>, I>(
  reducer: R,
  initializerArg: I & ReducerState<R>,
  initializer: (arg: I & ReducerState<R>) => ReducerState<R>,
): [ReducerState<R>, Dispatch<ReducerAction<R>>]

function useReducer<R extends Reducer<any, any>, I>(
  reducer: R,
  initializerArg: I,
  initializer: (arg: I) => ReducerState<R>,
): [ReducerState<R>, Dispatch<ReducerAction<R>>]

function useReducer<R extends Reducer<any, any>>(
  reducer: R,
  initialState: ReducerState<R>,
  initializer?: undefined,
): [ReducerState<R>, Dispatch<ReducerAction<R>>]
```

`useReducer` is pretty versatile. The key takeaway here is that while conventions are useful, understanding the API and its capabilities is more important.
