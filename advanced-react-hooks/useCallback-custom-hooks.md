# useCallback: custom hooks

## Background

### Memoization in general

Memoization: a performance optimization technique which eliminates the need to recompute a value for a given input by storing the original computation and returning that stored value when the same input is provided. Caching is a form of memoization.

```JSX
const values = {}
function addOne(num: number) {
  if (values[num] === undefined) {
    values[num] = num + 1 // <-- here's the computation
  }
  return values[num]
}
```

One other aspect of memoization is value referential equality.

```JSX
const dog1 = new Dog('sam')
const dog2 = new Dog('sam')
console.log(dog1 === dog2) // false
```

Even though those two dogs have the same name, they are not the same. However, we can use memoization to get the same dog:

```JSX
const dogs = {}
function getDog(name: string) {
  if (dogs[name] === undefined) {
    dogs[name] = new Dog(name)
  }
  return dogs[name]
}

const dog1 = getDog('sam')
const dog2 = getDog('sam')
console.log(dog1 === dog2) // true
```

You might have noticed that our memoization examples look very similar. Memoization is something you can implement as a generic abstraction:

```JSX
function memoize<ArgType, ReturnValue>(cb: (arg: ArgType) => ReturnValue) {
  const cache: Record<ArgType, ReturnValue> = {}
  return function memoized(arg: ArgType) {
    if (cache[arg] === undefined) {
      cache[arg] = cb(arg)
    }
    return cache[arg]
  }
}

const addOne = memoize((num: number) => num + 1)
const getDog = memoize((name: string) => new Dog(name))
```

Out abstraction only supports one argument, if you wannt to make it work for any type/number of arguments, knock yourself out.

### Memoization in React

Luckily, in React we don't have to implement a memoization abstraction. They made two for us! `useMemo` and `useCallback`.

`https://epicreact.dev/memoization-and-react/`

The dependency list of `useEffect`:

```JSX
React.useEffect(() => {
  window.localStorage.setItem('count', count)
}, [count]) // <-- that's the dependency list
```

Remember that dependency list is how React knows whether to call your callback (and if you don't provide one then React will call your callback every render). It does this to ensure that the side effect you're performing in the callback doesn't get out of sync with the state of the application.

But what happens if I use a function in my callback?

```JSX
const updateLocalStorage = () => window.localStorage.setItem('count', count)
React.useEffect(() => {
  updateLocalStorage()
}, []) // <-- what goes in that dependency list?
```

We could just put the `count` in the dependency list and that would actually/accidentally work, but what would happen if one day someone were change `updateLocalStorage`?

```JSX
- const updateLocalStorage = () => window.localStorage.setItem('count', count)

+ const updateLocalStorage = () => window.localStorage.setItem(key, count)
```

Would we remember to update the dependency list to include the `key`? Hopefully we would. But this can be a pain to keep track of dependencies. Especially if the function that we're using in our `useEffect` callback is coming to us fom props (in the case of a custom component) or arguments (in the case of a custom hook)

Instead, it would be much easier if we could just put the function itself in the dependency list:

```JSX
const updateLocalStorage = () => window.localStorage.setItem('count', count)
React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage]) // <-- function as a dependency
```

The problem with that though it will trigger the `useEffect` to run every render. This is because `updateLocalStorage` is defined inside the component function body. So it's re-initialized every render. Which means it's brand new every render. Which means it changes every render. Which means... you guessed it, our `useEffect` callback will be called every render!

### This is the problem `useCallback` solves. And here's how you solve it

```JSX
const updateLocalStorage = React.useCallback(
  () => window.localStorage.setItem('count', count),
  [count], // <-- yup! That's a dependency list!
)
React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage])
```

What that does is we pass React a function and React gives that same function back to us...

```JSX
// this is not how React actually implements this function. We're just imagining!
function useCallback(callback) {
  return callback
}
```

Here is the catch! On subsequent renders, if the elements in the dependency list are unchanged, instead of giving the same function back that we give to it, React will give us the same function it gave us last time. So imagine:

```JSX
// this is not how React actually implements this function. We're just imagining!
let lastCallback
function useCallback(callback, deps) {
  if (depsChanged(deps)) {
    lastCallback = callback
    return callback
  } else {
    return lastCallback
  }
}
```

So while we still create a new function every render (to pass to `useCallback`), React only gives us the new one if the dependency list changes.

`useCallback` is just a shortcut to using `useMemo` for functions:

```JSX
// the useMemo version:
const updateLocalStorage = React.useMemo(
  // useCallback saves us from this annoying double-arrow function thing:
  () => () => window.localStorage.setItem('count', count),
  [count],
)

// the useCallback version
const updateLocalStorage = React.useCallback(
  () => window.localStorage.setItem('count', count),
  [count],
)
```

A common question with this is: "Why don't we just wrap every function in `useCallback`"?
`https://kentcdodds.com/blog/usememo-and-usecallback`

And if the concept of a "closure" is new or confusing to you.
`https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures`

(Closures are one of the reasons it's important to keep dependency lists correct.)

## Other notes

### `useEffect` and `useCallback`

The use case for `useCallback` in the exercise is a perfect example of the types of problems `useCallback` is intended to solve. However the examples in these instructions are intentionally contrived. You can simplify things a great deal by not extracting the code from `useEffect` into functions that you then have to memoize with `useCallback`

`https://epicreact.dev/myths-about-useeffect/`

### `useCallback` use cases

The entire purpose of `useCallback` is to memoize a callback for use in dependency lists and props on memoized components (via `React.memo`, which you can learn more about from the performance workshop). The only time it's useful to use `useCallback` is when the function you're memoizing is used in one of those two situations.
