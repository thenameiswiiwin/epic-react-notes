# useState: greeting

## Background

`Hooks` used a special functions used to hold state.

- `React.useState`
- `React.useEffect`
- `React.useContext`
- `React.useRef`
- `React.useReducer`

Use to store data (`state`) or perform actions (`side-effects`).

### Example

```JSX
function Counter() {
    const [count, setCount] = React.useState(0)
    const increment = () => setCount(count++)
    return <button onClick={increment}>{count}</button>
}
```

`React.useState` is a function that accepts a single argument. That argument is the initial state for the instance of the component. In our case, the state will start as `0`

`React.useState` returns a pair of value. it does this by returning an array with two elelemts (and we use destructuring syntax to assign each of those values to distint variables). The first of the pair is the state value and the second is a function we can call to update the state.

`State` can be defined as: `data` that changes over time. When the button is clicked, our `increment` function will be called at which time we update the `count` by calling `setCount`.

When we call `setCount`, that tells React to re-render our component. When it does this. the entire `Counter` function is re-run, so when `React.useState` is called this time, the value we get back is the value that we called `setCount` with. And it continues like that until `Counter` is unmounted (removed from the application), or the user closes the application.
