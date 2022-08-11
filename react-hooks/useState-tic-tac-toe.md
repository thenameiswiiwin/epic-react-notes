# useState: tic tac toe

## Background

A `name` is one thing, but a real UI is a bit different. Often you need more than element of sate in your component, so you'll call `React.useState` more than once. Please note that each call `React.useState` in a given component will give you unique state and updater function.

## Exercise

We're going to build tic-tac-toe (with localStorage support!) If you've gone through React's official tutorial, this was lifted from that (except that exaple still uses classes).

You're going to need some managed state and some derived state:

- `Managed State`: State that you need to explicitly manage
- `Derived State`: State that you can calcualte based on other state

`squares` is the managed state and it's the state of the board in a single-dimensional array:

```
[
    'X', 'O', 'X',
    'X', 'O', 'O',
    'X', 'X', 'O'
]
```

This will start out as an empty array because it's the start of the game.

`nextValue` will be either the string 'X' or 'O' and is derived state which you can determine based on the value of `squares`. We can dtermine whose turn it is based on how many `"X"` `"O"` squares there are. We've written this out for you in a `calculateNextValue` function at the bottom of the file.

`winner` will be either the string `X` or `O` and is derived state which can also be determined based on the value of `squares` and we've provided a `calculateWinner` function you can use to get that value.
