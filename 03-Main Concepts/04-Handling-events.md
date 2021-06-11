# Handling events

- Event handling in React is very similar to handling events on DOM elements, only some syntax differences 
- React events are named using camelCase, rather than lowercase
- With JSX, one passes a function as the event handler instead of a string

```
// In plain HTML
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

```
// In React
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

- One cannot return `false` to present default behavior in React, one must call `preventDefault` explicitly

```
// In plain HTML
<form onsubmit="console.log("You clicked submit."); return false">
  <button type="submit">Submit</button>
</form>
```

```
// In React
function Form() {
  function handleSubmit(e) {
    e.preventDefault();
    console.log('You clicked submit.');
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Submit</button>
    </form>
  );
}
```

- `e` in this case is a synthetic event. React defines these synthetic events according to the W3C spec, so one doesn't have to worry about cross-browser compatibility
- When using React, one generally does not need to call `addEventListener` to add listeners to a DOM element after it's created. Instead, one can just provide a listener when the element is initially rendered
- With ES6 class, a common pattern is for an event handler to be a method on the class, see below

```
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isToggleOn: true };

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

- One should be careful about the meaning of `this` in JSX callbacks. In JS, class methods are not bound by default. So if one forgets to bind `this.handleClick` and pass it to `onClick`, `this` will be `undefined` when the function is actually called
- This isn't a React specific behavior, it's a part of how functions work in JS (in other words, it's because `this` context is determined by call-site). Generally, if one refer to a method without `()` after it, such as `onClick={this.handleClick}`, one should bind that method
- Two ways to get around `bind`:
1. If one is using the experimental public class fields syntax, one can use class fields to correctly bind callbacks

```
class LoggingButton extends React.Component {
  // THis syntax ensures `this` is bound within handleClick
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

- This syntax is enabled by default in CRA
- If one isn't using class field syntax, one can use an arrow function in the callback

```
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // This syntax ensures `this` is bound within handleClick
    return (
      <button onClick={() => this.handleClick()}>
        Click me
      </button>
    );
  }
}
```

- Issue with the above syntax is that a different callback is created each time the `LoggingButton` renders. In most cases, that's fine, but if this callback is passed as a prop to a lower component, those components might do an extra re-rendering
- Binding in the constructor or using the class fields syntax is recommended to avoid this sort of performance problem

## Passing arguments to event handles

- In a loop, it's common to pass an extra param to an event handler

```
// Say `id` is the row ID
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

- The above 2 lines are equivalent. They use arrow functions and Function.prototype.bind respectively
- In both cases, `e` argument representing the React event will be passed as a second argument after the ID. With an arrow function, one has to pass it explicitly, but with `bind` any further arguments are automatically forwarded