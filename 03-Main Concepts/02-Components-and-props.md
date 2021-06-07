# Components and props

- Components lets one split the UI into independent, reusable pieces, and think about each piece in isolation
- Conceptually, components are like JS functions. They accept arbitrary inputs known as **props** and return React elements describing what should appear on screen

## Function and class components

- Simplest way to define a component is with a JS function

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

- This function is a valid React component since it **accepts a single props** (**standing for properties**) object argument with data and **returns a React element**. Such components are known as function components because they're literally JS functions
- One can also use ES6 class to define a component

```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

- The two components above are equivalent from React's PoV

## Rendering a component

- So far, there has only been discussion around React elements that represent DOM tags

`const element = <div />;`

- However, elements can also represent user-defined components

`const element = <Welcome name="Sara" />;`

- When React sees an element representing a user-defined component, it passes **JSX attributes and children** to this component as a single object, aka **props**

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getDocumentById('root')
);
```

- Here's what's happening:
1. `ReactDOM.render()` is called with the Welcome element
2. React calls the Welcome component with `{name: 'Sara'}` as the props
3. The Welcome components returns a `<h1>Hello, Sara</h1>` element as the result
4. React DOM efficiently updates the DOM to match `<h1>Hello, Sara</h1>`

- Note: **always start component names with a capital letter**. React treats components starting with lowercase letters as DOM tags. E.g. `<div />` represents an HTML div tag, but `<Welcome />` represents a component and requires `Welcome` to be in scope

## Composing components

- Components can refer to other components in their output, allowing us to use the same component abstraction for any level of detail. E.g. A button, a form, a dialog which are all commonly expressed as components
- E.g. creating an App component that renders Welcome many times

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

- Typically, new React apps have a single top level App component. However, integrating React into an existing app might need to start from bottom-up with a small component like `Button` and work to the top of the view hierarchy

## Extracting component

```
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img
          className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

- This component can be tricky to change due to all of the nesting and it's also hard to reuse individual parts of it. Begin component extraction

```
function Avatar(props) {
  return (
    <img
      className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />
  );
}
```

- `Avatar` doesn't need to know that it's being rendered by `Comment`. Hence the reason behind a more generic name of `user` instead of `author`
- Naming props from the component's own PoV instead of the context of where it's being used is recommended

```
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name>
        {props.user.name}
      </div>
    </div>
  );
}
```

```
// Now Comment becomes
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date>
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

- Component extraction pays off in larger apps. Good rule of thumb is that if a part of UI is used several times (Button, Panel, Avatar), or is complex enough on its own (App, FeedStory, Comment), it's a good candidate to be extracted to a separate component

## Props are read-only

- When a component is defined from a function or class, it must never modify its own props

```
function sum(a, b) {
  return a + b;
}
```

- This is a pure function since it **doesn't attempt to change its inputs** and **always return the same result from the same inputs**
- A function becomes impure when it does change its own input:

```
function withdraw(account, amount) {
  account.total -= amount;
}
```

- React has a single strict rule: **all React components must act like pure functions with respect to their props**