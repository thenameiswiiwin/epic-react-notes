# useRef and useEffect: DOM interaction

## Background

Often when working with React you'll need to intergrate with UI libraries. Some of these need to work directly with the DOM. Remember that when you do: `<div>hi</div>` that's actually syntactic sugar for a `React.createElement` so you don't actually have access to DOM nodes in your function component. In fact, DOM node aren't created at all until the ReactDOM.render method is called. You function component is really just responsible for creating and returning React Elements and has nothing to do with the DOM in particular.

So to get access to the DOM, you need to ask React to give you access to a particular DOM node when it renders your component. The way this happens is through a special prop called `ref`.

### Example of using `ref` prop:

```JSX
function MyDiv() {
    cons myDivRef = React.useRef()
    React.useEffect(() => {
        // myDiv is the div DOM node!
        console.log(myDiv)
    }, [])
    return <div ref={myDivRef}>Hi</div>
}
```

After the component has been rendered, it's considered 'mounted.' That's when the React.useEffect callback is called and so by that point, the ref should have its `current` property set to the DOM node. So often you'll do direct DOM interactions/manipulations in the `useEffect` callback.
