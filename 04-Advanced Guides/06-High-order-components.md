# HOCs

- A high-order component (HOC) is an advanced technique in React for reusing component logic
- HOCs are not part of the React API, per se, instead, it's a pattern that emerges from React's compositional nature
- Concretely, a HOC is a function that takes a component and returns a new component

`const EnhancedComponent = highOrderComponent(WrappedComponent);`

- Whereas a component transforms props into UI, **a HOC transforms a component into another component**
- HOCs are common in third-party React libraries, such as Redux's `connect` and Relay's `createFragmentContainer`

## Use HOCs for cross-cutting concerns