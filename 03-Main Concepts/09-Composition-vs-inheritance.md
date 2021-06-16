# Composition vs inheritance

- React has a powerful composition model so it's recommended in using composition instead of inheritance to reuse code between components

## Containment

- Some components don't know their children ahead of time, especially common for components like `Sidebar` or `Dialog` that represent generic boxes
- It's recommended that such components use `children` prop to pass children elements directly to their output

```
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

```
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

- Anything inside the `<FancyBorder>` JSX tag gets passed into the `FancyBorder` component as a children prop. Since `FancyBorder` renders `{props.children}` inside a `<div>`, the passed elements appear in the final output
- This may be less common, but sometimes, one may need multiple "holes" in a component. In that case, one can come up with their own convention instead

```
function SplitPanel(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={<Contacts />}
      right={<Chat />}
    />
  );
}
```

- Once again, React elements like `<Contact />` and `<Chat />` are just objects, so one can pass them as props like any other data

## Specialization

- Sometimes, components are to be thought as special cases of other components. For example, one might say that a `WelcomeDialog` is a special case of `Dialog`
- This is also achieved by composition where a more "specific" component renders a more "generic" one and configures it with props

```
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog.message">
        {props.message}
      </P>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
    >
  );
}
```

```
// For components defined as classes
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = { login: '' };
  }

  render() {
    return (
      <Dialog
        title="Mars Exploration Program"
        message="How should we refer to you?"
      >
        <input value={this.state.login} onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>
          Sign Me Up!
        </button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({ login: e.target.value });
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```

## So what about inheritance?

- Facebook has yet to find any use cases where the usage of component inheritance hierarchies is recommended
- Props and composition give all the flexibility one needs to customize a component's look an behavior in an explicit and safe way
- If one wants to reuse non-UI functionality between components then it's suggested to extract it into a separate JS module. Components can then import it and use that function, object, or a class without extending it