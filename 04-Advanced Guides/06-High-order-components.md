# HOCs

- A high-order component (HOC) is an advanced technique in React for reusing component logic
- HOCs are not part of the React API, per se, instead, it's a pattern that emerges from React's compositional nature
- Concretely, a HOC is a function that takes a component and returns a new component

`const EnhancedComponent = highOrderComponent(WrappedComponent);`

- Whereas a component transforms props into UI, **a HOC transforms a component into another component**
- HOCs are common in third-party React libraries, such as Redux's `connect` and Relay's `createFragmentContainer`

## Use HOCs for cross-cutting concerns

- Components are the primary unit of code reuse in React, but one will find some patterns aren't a straightforward fit for traditional components
- E.g. say if one has a `CommentList` component that subscribes to an external data source to render a list of comments

```
class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" is some global data source
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // Subscribe to changes
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // Clean up listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // Update component state whenever the data source changes
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return(
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```

- One will later write a component for subscribing to a single blog post, which follows a similar pattern

```
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```

- `CommentList` and `BlogPost` aren't identical as they call different methods on `DataSource`, and they render different output, but much of their implementation is the same:
  - On mount, add a change listener to `DataSource`
  - Inside the listener, call `setState` whenever the data source changes
  - On unmount, remove the change listener
- One can imagine that in a large app, this same pattern of subscribe to `DataSource` and calling `setState` will occur over and over again
- Creating an abstraction will allow us to define this logic in a single place and share it across many components. Coming HOC to save the day
- One can write a function that creates components, like `CommentList` and `BlogPost`, that subscribe to `DataSource`
- The function will accept as one of its arguments a child component that receives the subscribed data as a prop. Let's call the function `withSubscription`

```
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
);
```

- The first param is the wrapped component and the second parameter retrieves the data we're interested in, given a `DataSource` and the current props
- When `CommentListWithSubscription` and `BlogPostWithSubscription` are rendered, `CommentList` and `BlogPost` will be passed a `data` prop with the most current data retrieved from `DataSource

```
// this function takes a component
function withSubscription(WrappedComponent, selectData) {
  // and returns another component
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // that takes care of the subscription
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnMount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

- Note that a HOC doesn't modify the input component, nor does it use inheritance to copy its behavior
- Instead, a HOC composes the original component by wrapping it in a container component
- A HOC is a pure function with zero side-effects
- The wrapped component receives all the props of the container, along with a new prop, `data`, which it uses to render its output
- The HOC isn't concerned with how or why the data is used, and the wrapped component isn't concerned with where the data came from
- Since `withSubscription` is a normal function, one can add as many or as few arguments as one would like
- E.g. one may make the name of the `data` prop configurable, to further isolate the HOC from the wrapped component
- Or one could accept an argument that configures `shouldComponentUpdate`, or one that configures the data source
- These are all possible because the HOC has full control over how the component is defined
- Like components, the contract between `withSubscription` and the wrapped component is entirely props-based
- This makes it easy to swap one HOC for a different one, as long as they provide the same props to the wrapped component
- This may be useful if one changes data-fetching libraries for example

## Don't mutate the original component, use composition

- Resist the temptation to modify a component's prototype (or otherwise mutate it) inside a HOC

```
function logProps(InputComponent) {
  InputComponent.prototype.componentDidUpdate = function(prevProps) {
    console.log('Current props: ', this.props);
    console.log('Previous props: ', prevProps);
  }

  // The fact that we're returning the original input is a hint that it has been mutated
  return InputComponent;
}

// EnhancedComponent will log whenever props are received
const EnhancedComponent = logProps(InputComponent);
```

- There are a few problems with this
  1. The input component cannot be reused separately from the enhanced component
  2. If one apply another HOC to `EnhancedComponent`, that also mutates `componentDidUpdate`, the first HOC's functionality will be overriden
  3. This HOC also won't work with function components, which do not have lifecycle methods
- Mutating HOCs are a leaky abstraction - the consumer must know how they are implemented in order to avoid conflict with other HOCs
- Instead of mutation, HOCs should use composition, by wrapping the input component in a container component

```
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('Current props: ', this.props);
      console.log('Previous props: ', prevProps);
    }

    render() {
      // Wraps the input component in a container, without mutating
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

- This new HOC has the same functionality as the mutating version while avoiding the potential for clashes
- It works equally well with class and function components, and because it's a pure function, it's composable with other HOCs, or even with itself
- One may have noticed similarities between HOCs and a pattern called **container components**

- Container components are part of a strategy of separating responsibility between high-level and low-level concerns
  - Containers manager things like subscription and state, and pass props to  components that handle things like rendering UI

- HOCs use container as part of their implementation
- One can think of HOCs as parameterized container component definitions

## Convention: Pass unrelated props through the wrapped component

- HOCs add features to a component
- They shouldn't drastically alter its contract
- It's expected that the component returned from a HOC has a similar interface to the wrapped component
- HOCs should pass through props that are unrelated to its specific concern
- Most HOCs contain a render method that looks something like this

```
render() {
  // Filter out extra props that are specific to this HOC and shouldn't be passed through
  const { extraProp, ...passThroughProps } = this.props;

  // Inject props into the wrapped component. These are usually state values or instance methods
  const injectProp = someStateOrInstanceMethod;

  // Pass prop to wrapped component
  return (
    <WrappedComponent
      injectedProps={injectedProp}
      {...passThroughProps}
    />
  );
}
```

- This convention helps ensure that HOCs are as flexible and reusable as possible

## Convention: Maximizing composability

- Not all HOCs look the same. Sometimes they only accept a single argument, the wrapped component

`const NavbarWithRouter = withRouter(Navbar);`

- Usually, HOCs accept additional arguments

`const CommentWithRelay = Relay.createContainer(Comment, config);`

- The most common signature for HOCs looks like this:

```
// React Redux's connect

const ConnectedComment = connect(commentListSelector, commentListActions)(CommentList);
```

```
// connect is a function that returns another function
const enhance = connect(commentListSelector, commentListActions);

// The returned function is a HOC, which returns a component that is connected to the Redux store
const ConnectedComment = enhance(CommentList);
```

- In other words, `connect` is a high-order function that returns a high-order component
- This form may seen confusing or unnecessary, but it has a useful property 
- Single-argument HOCs like the one returned by the `connect` function have the signature `Component => Component`
- Functions whose output type is the same as its input type are really easy to compose together

```
// Instead of doing this
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent));

// one can use a function composition utility compose(f, g, h) is the same as (...args) => f(g(h(...args)))
const enhance = compose(
  // These are both single-argument HOCs
  withRouter,
  connect(commentSelector)
);
const EnhancedComponent = enhance(WrappedComponent);
```

- (This same property also allows `connect` and other enhancer-style HOCs to be used as decorators, an experimental JS proposal)
- The `compose` utility function is provided by many third-party libraries including `lodash` and `Redux`

## Convention: Wrap the display name for easy debugging

- The container components created by HOCs show up in the React Dev Tools like any other component
- To ease debugging, choose a display name that communicates that it's the result of a HOC
- The most common technique is to wrap the display name of the wrapped component
- So if one HOC is named `withSubscription`, and the wrapped component's display name is `CommentList`, use the display name `WithSubscription(CommentList)`

```
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component { ... }
  WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;

  return WithSubscription;
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

## Caveats

- HOCs come with a few caveats that aren't immediately obvious

### Don't use HOCs inside the render method

- React's diffing algorothm called `Reconciliation` uses component identity to determine whether it should update the existing subtree or throw it away and mount a new one
- If the component returned from `render` is identical `===` to the component from the previous render, React recursively updates the subtree by diffing it with the new one
- If they're not equal, the previous subtree is unmounted completely
- Normally, one shouldn't need to think about this, but it matters for HOCs because it means one can't apply a HOC to a component within the render method of a component

```
render() {
  // A new version of EnhancedComponent is created on every render
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // That causes the entire subtree to unmount/remount each time
  return <EnhancedComponent />;
}
```

- Problem here isn't just performance - remounting a component causes the state of that component and all of its children to be lost
- Instead, apply HOCs outside of the component definition so that the resulting component is created only once. Then it's identity will be consistent across renders, which is usually what's wanted anyway
- In cases of dynamic HOC application, one can also do it inside a component's lifecycle method of its constructor

### Static method must be copied over

- Sometimes, it's useful to define a static method on a React component
- E.g. Relay container exposes a static method `getFragment` to facilitate the composition of GraphQL fragments
- When one applies a HOC to a component, though, the original component is wrapped with a container component
- The new component then won't have any of the static methods of the original component

```
// Define a static method
WrappedComponent.staticMethod = function() { ... }
// Now apply a HOC
const EnhancedComponent = enhance(WrappedComponent);

// The enhanced component has no static method
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

- To solve this, one could copy the method onto the container before returning it

```
function enhance(WrappedComponent) {
  class Enhance extends React.Component {...}

  // Must know exactly which methods to copy though :(
    Enhance.staticMethod = WrappedComponent.staticMethod;
    return Enhance;
}
```

- However, this requires one to know exactly which methods needs to be copied
- One can use `hoist-non-react-static` to automatically copy all non-React static methods

```
import hoistNonReactStatic from 'hoist-non-react-statics';

function enhance(WrappedComponent) {
  class Enhance extends React.Component { ... }
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

- Another possible solution is to export the static method separately from the component itself

```
// Instead of 
MyComponent.someFunction = someFunction;
export default MyComponent;

// export the method separately
export { someFunction };

// and in the consuming module, import both
import MyComponent, { someFunction } from "./MyComponent.js";

```

### Refs aren't passed through

- While the convention for HOCs is to pass through all props to the wrapped component, this doesn't work for refs
- That's because `ref` is not really a `prop` - like `key`, it's handled specially by React
- If one add a ref to an element whose component is the result of a HOC, the ref refers to an instance of the outermost container component, not the wrapped component
- Simply use `React.forwardRef` to solve this problem!