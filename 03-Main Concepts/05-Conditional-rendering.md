# Conditional rendering

- In React, one can create distinct components that encapsulate behaviours as needed. Then, one can choose to render only some of them, depending on the state of the application
- Conditional rendering in React works the same way conditions work in JS
- One can leverage JS operators like `if` or the `conditional operator` to create elements representing the current state, and let React update the UI to match them

```
function UserGreeting(props) {
  return <h1>Welcome back!</h1>;
}

function GuestGreeting(props) {
  return <h1>Please sing up.</h1>;
}
```

- Let's create a component to display either of these components depending on whether a user is logged in or not

```
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}

ReactDOM.render(
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
);
```

## Element variables

- One can use variables to store elements, which can help one conditionally render a part of the component while the rest of the output doesn't change

```
function LoginButton(props) {
  return (
    <button onClick={props.onClick}>
      Login
    </button>
  );
}

function LogoutButton(props) {
  return (
    <button onClick={props.onClick}>
      Logout
    </button>
  );
}
```

- Let's create a stateful component called `LoginControl`

```
class LoginControl extends React.Component {
  constructor(props) {
    super(props);
    this.handleLoginClick = this.handleLoginClick.bind(this);
    this.handleLogoutClick = this.handleLogoutClick.bind(this);
    this.state = { isLoggedIn: false };
  }

  handleLoginCLick() {
    this.setState({ isLoggedIn: true });
  }

  handleLogoutClick() {
    this.setState({ isLoggedIn: false });
  }

  render() {
    const isLoggedIn = this.state.isLoggedIn;
    let button;
    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />;
    } else {
      button = <LoginButton onClick={this.handleLoginClick} />;
    }

    return (
      <div>
        <Greeting isLoggedIn={isLoggedIn} />
        {button}
      </div>
    );
  }
}

ReactDOM.render(
  <LoginControl />,
  document.getElementById('root')
);
```

## Inline if with logical && operator

- One may embed expressions in JSX by wrapping them in curly braces, including JS' logical `&&` operator

```
function MailBox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}

const messages = ['React', "Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />
  document.getElementById('root')
);
```

- This works because in JS, `true && expression` always evaluates to `expression` while `false && expression` always evaluates to `false`
- So, if `true`, the element right after `&&` will appear in the output, if `false`, React ignores it and skips
- Note: returning a falsy expression will still cause the element after `&&` to be skipped but will return the falsy expression

```
render() {
  const count = 0;
  return (
    <div>
      { count && <h1>Messages: {count}</h1> }
    </div>
  );
  // Results in `<div>0</div>`
}
```

## Inline if-else with conditional operator

- Another method for conditionally rendering elements inline is to use the JS conditional operator `condition ? true : false`

```
render() {
  const isLoggedIn = this.state.isLoggedIn;

  return (
    <div>
      The user <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```

- Can also be used for larger expressions although it's less obvious

```
render() {
  const isLoggedIn = this.state.isLoggedIn;

  return (
    <div>
      {isLoggedIn
        ? <LogoutButton onClick={this.handleLogoutClick} />
        : <LoginButton onClick={this.handleLoginClick} />
      }
    </div>
  );
}
```

- Whenever conditions becomes too complex, it's might be a good idea to extract a component

## Preventing component from rendering

- In cases where one might want a component to hide itself even though it wa rendered by another component
- Do this by returning a `null` instead of its render output

```
function WarningBanner(props) {
  if(!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true};
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(state => ({
      showWarning: !state.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onCLick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

ReactDOM.render(
  <Page />,
  document.getElementById('root')
);
```

- Returning `null` from a component's `render` doesn't affect the firing of the component's lifecycle methods. E.g. `componentDidUpdate` will still be called