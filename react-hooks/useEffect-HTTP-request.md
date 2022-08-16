# useEffect: HTTP requests

## Background

HTTP requrest are another common side-effect that we need to do in applications. This is no differentfrom the side-effects we need to apply to a rendered DOM or when interacting with browser APIs like localStorage. In all these cases, we do that within a `useEffect` hook callback. This hook allows us to ensure that whenever certain changes take place, we apply the side-effects based on those changes.

One important thing to note about the `useEffect` hook is that you cannot return anything other than the cleanup function. This has interesting implications with regfard to async/await synctax:

```JSX
// this does not work, don't do this:
React.useEffect(async () => {
    const result = await doSomeAsyncThing()
    // do something with the result
})
```

The reason this doesn't work is because when you make a function async, it automatically returns a promise (whether you're not returning anything at all, or explicitly returning a function). This is due to the sematics of async/await synctax. So if you want to use aync/await, the best way to do that is like so:

```JSX
React.useEffect(() => {
    async function effect() {
        const result = await doSomeAsyncThing()
        // do something with the result
    }
})
```

This ensures that you don't return anything but a cleanup function.

It's typically just easier to extract all the async code into a utility function which you call and then use the promise-based `.then` method instead of using async/await syntax:

```JSX
React.useEffect(() => {
    doSomeAsyncThing().then(result => {
        // do something with the result
    })
})
```

## 💯 Handle Errors

Unfortunately, sometimes things go wrong and we need to handle errors when they do so we can show the user useful information. Handle that error and render it out like so:

```JSX
<div>
    There was an error:{' '}
    <pre style={{whiteSpace: 'normal'}}>{error.message}</pre>
</div>
```

You can make an error happen by typing an incorrect pokemon name into the input.

How to handle promise errors?

There is 2 ways:

```JSX
// Option 1: using .catch
fetchPokemon(pokemonName)
    .then(pokemon => setPokemon(pokemon))
    .catch(error => setError(error))

// Option 2: using the second argument to .then
fetchPokemon(pokemonName).then(
    pokemon => setPokemon(pokemon),
    error => setError(error),
)
```

These are funtionally equivalent for our purposes, but they are sematically different in general.

Using `.catch` means that you'll handle an error in the `fetchPokemon` promise, but you'll also handle an error in the `setPokemon(pokemon)` call as well. This is due to the ematics of how promises work.

Using the second argument to `.then` means that you will catch an error that happends in `fetchPokemon` only. In this case, calling `setPokemon` would not throw an error (React handles errors and we have an API to catch those which we'll use later).

## 💯 Use a Status

Our logic for what to show the user when is kind of convoluted and requires that we be really careful about which state we set and when.

We could make things much simpler by having some state to set the explicit status of our component. Our component can be in the following "states":

- `idle`: no request made yet
- `pending`: request started
- `resolved`: request sucessful
- `rejected`: request failed

Try to use a status state by setting it to these string values rather than relying on existing state or booleans.

Learn more about this concept here: `https://kentcdodds.com/blog/stop-using-isloading-booleans`

## 💯 store the state in an object

You'll notice that we're calling a bunch of state updaters in a row. This is normally not a problem, but each call to our state updater can result in a re-render of our component. React normally batches these calls so you only get a single re-render, but it's unable to do this in an asynchronous callback (like our promise success and error handlers).

So you might notice that if you do this:

```JSX
setStatus('resolved')
setPokemon(pokemon)
```

You'll get an error indicating that you cannot read `image` of `null`. This is because the `setStatus` call results in a re-render that happends before the `setPokemon` happens.

> but it's unable to do this in an asynchronous callback

This is no longer the case in React 18 as it supports automatic batching for asynchronous callback too.

Learn more about this concept here: `https://reactjs.org/blog/2022/03/29/react-v18.html#new-feature-automatic-batching`

Still it is better to maintain closely related states as an object rather than maintaining them using individual useState hooks.

Learn more about this concept here: `https://kentcdodds.com/blog/should-i-usestate-or-usereducer#conclusion`

In the future, you'll learn about how `useReducer` can solve this problem really elegantly, but we can still accomplish this by storing our state as an object that has all the properties of state we're managing.

See if you can figure out how to store all of your state in a single object with a single `React.useState` call so I can update my state like this:

```JSX
setState({status: 'resolved', pokemon})
```

## 💯 create an ErrorBoundary component

We've already solved the problem for errors in our request, we're only handling that one error. But there are a lot of different kinds of errors that can happen in our applications.

No matter how hard you try, eventually your app code just isn’t going to behave the way you expect it to and you’ll need to handle those exceptions. If an error is thrown and unhandled, your application will be removed from the page, leaving the user with a blank screen.

Luckily for us, there’s a simple way to handle errors in your application using a special kind of component called an `Error Boundary`. Unfortunately, there is currently no way to create an Error Boundary component with a function and you have to use a class component instead.

- `Error Boundary`: `https://reactjs.org/docs/error-boundaries.html`

Unfortunately, there is currently no way to create an Error Boundary component with a function and you have to use a class component instead.

## 💯 re-mount the error boundary

You might notice that with the changes we've added, we now cannot recover from an error. For example:

- 1. Type an incorrect pokemon
- 2. Notice the error
- 3. type a correct pokemon
- 4. Notice it doesn't show the new pokemon's information

The reason this is happening is because the `error` that's stored in the internal state of the `ErrorBoundary` component isn't getting reset, so it's not rendering the `children` we're passing to it.

So what we need to do is reset the ErrorBoundary's `error` state to `null` so it will re-render. But how do we access the internal state of our `ErrorBoundary` to reset it? We'll, there are a few ways we could do this by modifying the `ErrorBoundary`, but one thing you can do when you want to reset the state of a component, is by providing it a `key` prop which can be used to unmount and re-mount a component.

`key={pokemonName}`

## 💯 use react-error-boundary

As cool as our own `ErrorBoundary` is, I'd rather not have to maintain it in the long-term. Luckily for us, there's an npm package we can use instead and it's already installed into this project. It's called `react-error-boundary`.

`https://github.com/bvaughn/react-error-boundary`

Go ahead and give that a look and swap out our own `ErrorBoundary` for the one from `react-error-boundary`.
