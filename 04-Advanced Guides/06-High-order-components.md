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