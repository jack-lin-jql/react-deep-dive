# Forwarding refs

- Ref forwarding is a technique for automatically passing a `ref` through a component to one of its children
- This is typically not necessary for most components in the application, however, it can be useful for some kinds of components, especially in reusable component libraries

## Forwarding refs to DOM components

- Consider a `FancyButton` component that renders the native `button` DOM element

```
function FancyButton(props) {
  return (
    <button className="FancyButton">
      {props.children}
    </button>
  );
}
```

- React component hide their implementation details, including their rendered output
- Other components using FancyButton usually will not need to obtain a ref to the inner `button` DOM element
- This is good since it prevents components from relying on each other's DOM structure too much
- Although such encapsulation is desirable for application-level components like `FeedStory`, `Comment`, it can be convenient for highly reusable "leaf" components like `FancyButton` or `MyTextInput`
- These components tend to be used throughout the application in a similar manner as a regular DOM `button` and `input`, and accessing their DOM nodes may be unavoidable for managing focus, selection, or animations

- Ref forwarding is an opt-in feature that lets some components take a ref they receive, and pass it further down (in other words, "forward it") to a child

- See below, `FancyButton` uses `React.forwardRef` to obtain the `ref` passed to it, and then forward it to the DOM `button` that it renders

```
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// One can now get a ref directly to the DOM button
const ref = React.creteRef();
<FancyButton ref={ref}> Click me!</FancyButton>
```

- This way, components using `FancyButton` can get a ref to the underlying `button` DOM node and access it if necessary - just like if they used a DOM `button` directly
- Here's the breakdown of the example
  1. Created a React ref by calling `React.createRef` and assigned it to a `ref` variable
  2. Pass our `ref` down to `<FancyButton ref={ref}>` by specifying it as a JSX attribute
  3. React passes the `ref` to the `(props, ref) => ...` function inside `forwardRef` as a second argument
  4. Forward this `ref` argument down to `<button ref={ref}>` by specifying it as a JSX attribute
  5. When the ref is attached, `ref.current` will point to the `<button>` DOM node
- Note: the second `ref` argument only exists when one define a component with `React.forwardRef` call. Regular function or class components don't receive the `ref` argument, and is not available in props either
- Ref forwarding isn't limited to DOM components, one can forward refs to class component instances, too

## Note for component library maintainers

- When one start using forwardRef in a component library, one should treat it as a breaking change and release a new version of the library
- This is because your library likely has an observably different behavior (such as what refs get assigned to, and what types are exported), and this can break apps and other libraries that depend on the old behavior
- Conditionally applying `React.forwardRef` when it exists is also not recommended for the same reason: it changes how a library behaves and can break users' apps when they upgrade React itself

## Forwarding refs in HOC

- This technique can also be particularly useful with HOCs

```
function logProps(WrappedComponent) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      return <WrappedComponent {...this.props} />;
    }
  }

  return LogProps;
}
```

- The logProps HOC passes all `props` through to the component it wraps, so the rendered output will be the same
- E.g., use this to log all props that get passed to our "fancy button" component

```
class FancyButton extends React.Component {
  focus() {
    // ...
  }

  // ...
}

// Rather than exporting FancyButton, we export LogProps
// It will render a FancyButton though
export default logProps(FancyButton);
```

- There's one caveat with the above example: refs will not get passed through
- This is because `ref` is not a prop, like `key`, it's handled differently by React
- If one adds a ref to a HOC, the ref will refer to the outermost container component, not the wrapped component
- This means refs intended for our `FancyButton` component will actually be attached to the `LogProps` component

```
import FancyButton from "./FancyButton";

const ref = React.createRef();

// The FancyButton component we imported is the LogProps HOC. Even though the rendered output will be the same, our ref will point to LogProps instead of the inner FancyButton
// This means we can't call e.g. ref.current.focus()
<FancyButton
  label="Click Me"
  handleClick={handleClick}
  ref={ref}
/>;
```

- Fortunately, we can explicitly forward refs to the inner `FancyButton` component using the `React.forwardRef` API
- `React.forwardRef` accepts a render function that receives `props` and `ref` params and returns a React node

```
function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;

      // Assign the custom prop "forwardedRef" as a ref

      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  // Note the second param "ref" provided by React.forwardRef. We can pass it along to LogProps as a regular prop, e.g. "forwardedRef"
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}
```

## Displaying a custom name in DevTools

- `React.forwardRef` accepts a render function. React DevTools uses this function to determine what to display for the ref forwarding component
- E.g. the following component will appear as "ForwardRef" in the DevTools

```
const WrappedComponent = React.forwardRef((props, ref) => {
  return <LogProps {...props} forwardRef={ref} />;
});
```

- If one names the function, DevTools will also include its name (e.g. "ForwardRef(myFunction)")

```
const WrappedComponent = React.frowardRef(
  function myFunction(props, ref) {
    return <LogProps {...props} forwardRef={ref} />;
  }
);
```

- One can event set the function's `displayName` property to include the component one is wrapping

```
function logProps(Component) {
  class LogProps extends React.Component {
    // ...
  }

  function forwardRef(props, ref) {
    return <LogProps {...props} forwardedRef={ref} />;
  }

  // Give this component a more helpful display name in DevTools.
  // E.g. "ForwardRef(logProps(MyComponent))"
  const name = Component.displayName || Component.name;
  forwardRef.displayName = `logProps(${name})`;

  return React.forwardRef(forwardRef);
}
```