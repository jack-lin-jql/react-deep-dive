# Refs and the DOM

- Refs provide a way to access DOM nodes or React elements created in the render method
- In the typical React dataflow, `props` are the only way the parent components interact with their children
  - To modify a child, one re-render it with new props
- However, there are a few cases where on need to imperatively modify a child outside of the typical dataflow
- The child to be modified could be an instance of a React component, or it could be a DOM element. For both of these cases, React provides an escape hatch

## When to use refs

- A few good use cases:
  - Managing focus, text selection, or media playback
  - Triggering imperative animations
  - Integrating with third-party DOM libraries
- Avoid using refs for anything that can be done declaratively
- E.g. instead of exposing `open()` and `close()` methods on a `Dialog` component, pass an `isOpen` prop to it

## Don't overuse refs

- One's first inclination may be to use refs to "make things happen" in their app
- If this is the case, take a moment and think more critically about where state should be owned in the component hierarchy
- Often, it becomes clear that the proper place to "own" that state is at a higher level in the hierarchy, so simply lift the state up
- Note: It's recommended for earlier versions (before 16.3) of React to use callback refs instead of `React.createRef()`
