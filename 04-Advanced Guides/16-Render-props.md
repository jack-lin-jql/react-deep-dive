# Render props

- "Render prop" refers to a technique for sharing code between React components using a prop whose value is a function
- A component with a render prop takes a function that returns a React element and calls it instead of implementing its own render logic

```
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}>
```

- Libraries that use render props include `React Router`, `Downshift`, and `Formik`

## Use render props for cross cutting concerns

- Components are the primary unit of code reuse in React, but it's not always obvious how to share the state or behavior that one component encapsulates to other components that need that same state
- E.g., the following component tracks the mouse position in a web app:

```
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        <h1>Move the mouse around!</h1>
        <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
      </div>
    );
  }
}
```

- As the cursor moves around the screen, the component displays its (x, y) coordinates in a `<p>`
- Now the question is: how can we reuse this behavior in another component?
- In other words, if another component needs to know about the cursor position, can we encapsulate that behavior so that we can easily share it with that component?
- Since components are the basic unit reuse in React, let's try refactoring the code a bit to use a `<Mouse>` component that encapsulates the behavior needed to reuse elsewhere

```
// The <Mouse> component encapsulates the behavior needed
class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        {/* but how do we render something other than a <p>? */}
        <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <>
        <h1>Move the mouse around!</h1>
        <Mouse />
      </>
    );
  }
}
```

- Now the `<Mouse>` component encapsulates all behavior associated with listening for `mousemove` events and storing the (x, y) position of the cursor, but it's not yet truly reusable
- E.g. let's say we have a `<Cat>` component that renders the image of a cat chasing the mouse around the screen. We might use a `<Cat mouse={{ x, y }}>` prop to tell the component that coordinates of the mouse so it knows where to position the image on the screen
- As a first pass, one might try rendering the `<Cat>` inside `<Mouse>`'s render method like the following:

```
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.png" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class MouseWithCat extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        {
          /*
            One could just swap out the <p> for a <Cat> here but then we would need to create a separate <MouseWithSomethingElse> component every time we need to use it, so <MouseWithCat> isn't really reusable yet
          */
        }
        <Cat mouse={this.state} />
      /div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <MouseWithCat />
      </div>
    );
  }
}
```

- This approach works for the specific use case, but it does not achieve the objective of truly encapsulating the behavior in a reusable way
- Now, every time one wants the mouse position for a different use case, one would have to create a new component that renders something specifically for that use case
- Here's where the render prop comes in: instead of hard-coding a `<Cat>` inside a `<Mouse>` component, and effectively changing its rendered output, one can provide `<Mouse>` with a function prop that it uses to dynamically determine what to render-a render prop

```
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.png" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        {
          /*
            Instead of providing a static representation of what <Mouse> renders, use the `render` prop to dynamically determine what to render.
          */
        }
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )} />
      </div>
    )
  }
}
```

- Now instead of effectively cloning the `<Mouse>` component and hard-coding something else in its `render` method to solve for a specific use case, we provide a `render` prop that `<Mouse>` can use to dynamically determine what it renders
- More concretely, a render prop is a function prop that a component uses to know what to render
- This technique makes the behavior that's needed to share extremely portable
- To get that behavior, render `<Mouse>` with a `render` prop that tells it what to render with the current (x, y) of the cursor
- One interesting thing to note about render props is that one can implement most HOCs using a regular component with a render prop
- E.g. if one prefer to have `withMouse` HOC instead of a `<Mouse>` component, one could easily create one using a regular `<Mouse>` with a render prop:

```
// If one really wants a HOC for some reason, one can easily create one using a regular component with a render prop!
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )}>
      );
    }
  }
}
```

- So using a render prop makes it possible to use either pattern

## Using props other than render

- It's important to remember that just because that pattern is called "render props" one doesn't have to use a prop named after `render` to use this pattern
- In fact, any prop that is a function that a component uses to know what to render is technically a "render prop"
- Although the examples above use `render`, one could just as easily use the `children` prop

```
<Mouse children={mouse => (
  <p>The mouse position is {mouse.x}, {mouse.y}</p>
)}>
```

- And remember, that `children` doesn't actually need to be named in the list of "attributes" in one's JSX element. Instead, one can put it directly inside the element

```
<Mouse>
  {mouse => (
    <p>The mouse position is {mouse.x}, {mouse.y}</p>
  )}
</Mouse>
```

- Since this technique is a little unusual, one will probably want to explicitly state that `children` should be a function in the `propTypes` when designing an API like this

```
Mouse.propTypes = {
  children: PropTypes.function.isRequired
}
```

## Caveats

- Be careful when using render props with React.PureComponent
- Using a render prop can negate the advantage that comes from using `React.PureComponent` if one create the function inside a `render` method
- This is because the shallow prop comparison will always return false for new props and each `render` in this case will generate a new value for the render prop
- E.g. continuing with the `<Mouse>` component from above, if `Mouse` were to extends `React.PureComponent` instead of `React.Component`, the example would look like this:

```
class Mouse extends React.PureComponent {
  // Same implementation as above
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        {
          /*
            This is bad, the value of the render prop will be different on each render
          */
        }
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )} />
      </div>
    );
  }
}
```

- Here, each time `<MouseTracker>` renders, it generates a new function as the value of the `<Mouse render>` prop, thus negating the effect of `<Mouse>` extending `React.PureComponent` in the first place
- To get around this problem, one can something define the prop as an instance method like so:

```
class MouseTracker extends React.Component {
  // Defined as an instance method, `this.renderTheCat` always refers to *same* function when we use it in render
  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```

- In cases where one cannot define the prop statically (e.g. because one needs to close over the component's props and/or state) `<Mouse>` should extend `React.Component` instead