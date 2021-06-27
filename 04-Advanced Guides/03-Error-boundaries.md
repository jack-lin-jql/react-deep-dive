# Error boundaries

- In the past, JS errors inside components used to corrupt React's internal states and cause it to emit cryptic errors on next renders
- These errors were always caused by an earlier error in the application code, but React did not provide a way to handle them gracefully in components, and couldn't recover from them

## Introducing error boundaries

- A JS error in a part of the UI shouldn't break the whole app
- So, React 16 introduced a new concept of error boundary
- Error boundaries are React components that **catch JS errors anywhere in their child component tree, logs those errors, and display a fallback UI** instead of the component tree that crashed
- Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them
- Note: error boundaries do not catch errors for
  - Event handlers
  - Async code (e.g. setTimeout, or requestAnimationFrame callbacks)
  - Server side rendering
  - Error thrown in the error boundary itself (rather than its children)