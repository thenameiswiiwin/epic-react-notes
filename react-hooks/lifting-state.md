# Lifting State

## Background

Sharing state between two siblings components.

`https://reactjs.org/docs/lifting-state-up.html`

`Lifting the State` basically amounts to finding the lowest common parent shared between the two components and placing the state management there, and then passing the state and a mechanism for updating that state down into the components that need it.

Often, several components need to reflect the same changing data. We recommed lifting the shared state up to their closest common ancestor.

## Colocating State

Pushing state back down.

`https://kentcdodds.com/blog/colocation`

### The principle

The concept of co-location can be boiled down to this fundamental princible:

`Place code as close to where it's relevant as possible`

you might also say: "Things that change together should be located as close as reasonable."

`https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster`

One of the leading causes to slow React applications is global state, especially the rapidly changing variety.
