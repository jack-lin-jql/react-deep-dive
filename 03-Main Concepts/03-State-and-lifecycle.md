# State and lifecycle 

- From previous, one way to update the UI was through `ReactDOM.render()` to change the rendered output

```
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

- To make the `Clock` component truly reusable and encapsulated, it'll have to setup its own timer and update itself every second

```
// Start with encapsulating Clock
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock data={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

- The above is missing the fact that `Clock` should set up a timer and updates the UI every second as its own implementation detail
- Ideally, the following should only be written once and have `Clock` update itself

```
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

- Here's where **state** comes to save the day
- State is similarly to props, but it's private and fully controlled by the component

## Converting a function to a class

- To convert a function component into a class, here's what one needs:
1. Create a ES6 class with the same name that extends React.Component
2. Add a single empty method to it called `render()`
3. Move the body of the function into the `render()` method
4. Replace `props` with `this.props` in the `render()` body
5. Delete the remaining empty function declaration

```
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}</h2>
      </div>
    );
  }
}
```

- The `render` method will be called each time an update happens, but as long as we render `<Clock />` into the same DOM node, only a single instance of the Clock class will be used, allowing us to use additional features such as local state and lifecycle methods