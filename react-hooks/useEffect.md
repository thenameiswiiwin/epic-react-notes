# useEffect: persistent state

## Background

`React.useEffect` allows you to run some custom code after React renders (and re-renders) your component to the DOM. It accepts a callback function which React will call after the DOM has been updated:

```JSX
React.useEffect(() => {
    // your side-effect code here
    // this is where you can make HTTP requess or interact with browser APIs.
})
```

`src/examples/hook-flow.png` - example of the timing of when yoiur functions are run.


## Lazy State Initialization

```JSX
const [name, setName] = React.useState(window.localStorage.getItem('name') ?? initialName)

React.useEffect(() => {
window.localStorage.setItem('name', name)
})
```
Right now, everytime our component function is run, our function reads from localStorage. This is problematic because it could be a performance bottleneck (reading from localStorage can be slow). And what's more we only actually need to know the value from localStorage the first time this component is rendered! So the additional reads are wasted effort.

To avoid this problem, React's useState hook allows you to pass a function instead of the actual value, and the it will only call that function to get the state value when the component is rendered the first time. So you can go from this:

`React.useState(someExpensiveComputation())`

To this:

`React.useState(() => someExpensiveComputation())`

and the `someExpensiveComputation` function will only be called when it's needed!

```JSX
const [name, setName] = React.useState(
    () => window.localStorage.getItem('name') ?? initialName,
)
```

## Effect Dependencies

The callback we're passing to `React.useEffect` is called after every render of our component (including re-renders). This is exactly what we want because we want to make sure that the `name` is saved into localStorage whenever it changes, but there are various reasons a component can be re-rendered (for example, when a parent component in the application tree gets re-rendered)

Really, we only want localStorage to get updated when the `name` state actually changes. It doesn't need to re-run any other time. Luckily for us, `React.useEffect` allows you to pass a second argument called the `dependency array` which signals to React that your effect callback function should be called when (and only when) those dependencies change. So we can use this to avoid doing unnecessary work!

```JSX
React.useEffect(() => {
    window.localStorage.setItem('name', name)
}, [])
```

## Custom Hook

The best part of hooks is that if you find a bit of logic inside your component function that you think would be useful elsewhere, you can put that in another function and call it from the components that need it (just like regular JavaScript). These functions you create are called `Custom Hooks`.

```JSX
function useLocalStorageState(key, defaultValue = '') {
    const [state, setState] = React.useState(
        () => window.localStorage.getItem(key) ?? defaultValue,
    )

    React.useEffect(() => {
        window.localStorage.setItem(key, state)
    }, [key, state])

    return [state, setState]
}

function Greeting({initialName = ''}) {
    const [name, setName] = useLocalStorageState('name', initialName)
    .......
)
```

## Flexible localStorage Hook

Take the custom `useLocalStorageState` hook and make it generic enough to support any data type (remember, you have to serialize objects to strings... use `JSON.stringigy` and `JSON.parse`).

### Notes

If you'd like to learn more about when different hooks are called and the order in which they're called, then open up `src/examples/hook-flow.png` and `src/examples/hook-flow.js`

```JSX
function useLocalStorageState(
    key,
    defaultValue = '',
    {serialize = JSON.stringify, deserialize = JSON.parse} = {},
) {
    const [state, setState] = React.useState(() => {
    const valueInLocalStorage = window.localStorage.getItem(key)
    if (valueInLocalStorage) {
      try {
        return deserialize(valueInLocalStorage)
      } catch (error) {
        window.localStorage.removeItem(key)
      }
    }
    return typeof defaultValue === 'function' ? defaultValue() : defaultValue
    })

    const prevKeyRef = React.useRef(key)

    React.useEffect(() => {
    const prevKey = prevKeyRef.current
    if (prevKey !== key) {
      window.localStorage.removeItem(prevKey)
    }
    prevKeyRef.current = key
    window.localStorage.setItem(key, serialize(state))
    }, [key, state, serialize])

    return [state, setState]
}
```
