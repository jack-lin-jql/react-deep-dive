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
- A class component becomes an error boundary if it defines either (or both) of the lifecycle method `static getDerivedStateFromError()` or `componentDidCatch()`
- Use `static getDerivedStateFromError()` to render a fallback UI after an error has been thrown
- Use `componentDidCatch()` to log error information

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    //Update state so the next render shows the fallback UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // One can also log the error to an error reporting service
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // One can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

```
// Use as a regular component
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

- Error boundaries work like JS `catch{}` block but for components
- Only class components can be error boundaries
- In practice, most of the time, one will want to declare an error boundary component once and use it through out the application
- Note that **error boundaries only catch errors in the components below them in the tree**
- An error boundary can't catch an error within itself
- If an error boundary fails trying to render the error message, the error will propagate to the closest error boundary above it
- This, too, is similar to how catch {} block works in JS

## Where to place error boundaries

- The granularity of error boundaries is up to you
- One may wrap top-level route components to display a "Something went wrong" message to the user, just like how server-side frameworks often handle crashes
- One may also wrap individual widgets in an error boundary to protect them from crashing the rest of the application

## New behavior for uncaught errors

- As of React 16, errors that weren't caught by any error boundary will result in unmounting of the whole React component tree
- In the author's experience, it's worse to leave corrupted UI in place than to completely remove it
- E.g. Messenger leaving the broken UI visible could lead to someone sending a message to the wrong person
- E.g. Payments app to display a wrong amount than to render nothing
- Adding error boundaries lets one provide better UX when something goes wrong
- Apps like FB Messenger wraps contents of the sidebar, the info panel, the convo log, and the message input into separate error boundaries, so if some components in one of these UI areas crashes, the rest of them remain interactive
- Reporting services should be used to report unhandled exceptions as they happen in production

## Component stack traces

- React 16 prints all errors that ocurred during rendering to the console in development, even if the application accidentally swallows them
- In addition to the error message and the JS stack, it also provides **component stack traces**
- Now one can see where exactly in the component tree the failure has occurred
- One can see the filenames and line numbers in the component stack trace (this works by default in CRA projects)
- If not using CRA, one can add [this plugin](https://www.npmjs.com/package/@babel/plugin-transform-react-jsx-source) manually to Babel 
- Note that it's intended only for development and must be disabled in production
- Note: Component names displayed in the stack traces depend on the `Function.name` property
- If supporting older browsers and devices which may not provide this natively (e.g. IE 11), consider including a `Function.name` polyfill in the bundled application, such as [function.name-polyfill](https://github.com/JamesMGreene/Function.name). One may also explicitly set the display name for all components

## How about try/catch?

- try/catch only works for imperative code

```
try {
  showButton();
} catch (error) {
  // ...
}
```

- However, React components are declarative and specify what should be rendered

`<Button />`

- Error boundaries preserve the declarative nature of React and behave as one would expect
- E.g. if an error occurs in a `componentDidUpdate` method caused by a `setState` somewhere deep in the tree, it'll still correctly propagate to the closest error boundary

## How about event handlers?

- Error boundaries do not catch error inside event handlers
- React doesn't need error boundaries to recover from errors in event handlers
- Unlike the render method and lifecycle methods, the events handlers don't happen during rendering, so if they throw, React still knows what to display on the screen
- If one need to catch an error inside an event handler, use the regular JS try/catch statement

```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { error: null };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    try {
      // Do something that could throw
    } catch (error) {
      this.setState({ error });
    }
  }

  render() {
    if (this.state.error) {
      return <h1>Caught an error.</h1>;
    }

    return <button onClick={this.handleCLick}>Click Me</button>;
  }
}
```