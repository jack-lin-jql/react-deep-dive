# Passing functions to components

## How do I bind a function to a component instance?

- There are several ways to make sure functions have access to component attributes like `this.props` and `this.state`

### Bind in construction (ES15)

```js
class Foo extends Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    console.log('...');
  }

  render() {
    return ... onClick={this.handleClick}...;
  }
}
```

### Class properties (stage 3 proposal)

```js
class Foo extends Component {
  handleClick = () => {
    ...
  }

  render() {
    return ...;
  }
}
```

### Bind in render

```js
class Foo extends Component {
  handleClick() {
    ...
  }

  render() {
    // Here, bind creates a new function on each component render which can be detrimental on performance
    return <button onClick={this.handleClick.bind(this)}>Click Me</button>;
  }
}
```



### Arrow function in render

```js
class Foo extends Component {
  handleClick() {
    ...
  }

  render() {
    return <button onClick={() => this.handleClick()}>Click me</button>;
  }
}
```

- Here, arrow function in render also creates a new function each time the component renders, which may break optimization based on strict identity comparison

## Is it ok to use arrow functions in render methods?

- Yes, generally, it's okay. It's also often the easiest way to pass parameters to callback functions

## Why is binding necessary

- In JS, the following aren't equivalent

`obj.method();`

```js
var method = obj.method;
method;
```

- Binding methods help ensure that the second snippet works the same way as the first
- With React, typically, one would only need to bind the methods passed to other components
  - E.g. `<button onClick={this.handleClick}>` passes `this.handleClick` so one would want to bind it
  - But it's unnecessary to bind the `render` method or the lifecycle methods as they don't get passed to other components

 ## How does one pass a param to an event handler or callback?

 `<button onClick=() => this.handleClick(id)} />`

 ### E.g. passing params using arrow functions 

 ```js
 const A = 65;

 class Alphabet extends React.Component {
   constructor(props) {
     super(props);
     this.state = {
       justClicked: null,
       letters: Array.from({length: 26}, (_, i) => String.fromCharCode(A+i))
     };
   }

   handleClick(letter) {
     this.setState({ justClicked: letter });
   }

   render() {
     return (
       <div>
        Just clicked: {this.state.justClicked}
        <ul>
          {this.state.letters.map(letter =>
            <li key={letter} onClick={() => this.handleClick(letter)}>
              {letter}
            </li>
          )}
        </ul>
       </div>
     );
   }
 }
 ```

 ### E.g. passing params using data-attributes

 - One can alternatively use DOM API to store data needed for event handlers
 - Consider this if needing to optimize large number of elements or have a render tree that relies on React.PureComponent equality checks

 ```js
 const A = 65 // ASCII character code

class Alphabet extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
    this.state = {
      justClicked: null,
      letters: Array.from({length: 26}, (_, i) => String.fromCharCode(A + i))
    };
  }

  handleClick(e) {
    this.setState({
      justClicked: e.target.dataset.letter
    })
  }

  render() {
    return (
      <div>
        Just clicked: {this.state.justClicked}
        <ul>
          {this.state.letters.map(letter =>
            <li key={letter} data-letter={letter} onClick={this.handleClick}>
              {letter}
            </li>
          )}
        </ul>
      </div>
    );
  }
}
 ```

 ## How can I prevent a function from being called too quickly or too many times in a row?

 - If one has an event handler such as `onClick` or `onScroll` and want to prevent the callback from being fired too quickly then one can limit the rate at which callback is executed
  - Throttling: sample changes based on a time based frequency (e.g. _.throttle)
  - Debouncing: publish changes after a period of inactivity (e.g. _.debounce)
  - requestAnimationFrame throttling: sample changes based on requestAnimationFrame

### Throttle

- Prevents a function from being called more than once in a given window of time

```js
import throttle from "lodash.throttle";

class LoadMoreButton extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
    this.handleClickThrottled = throttle(this.handleClick, 1000);
  }

  componentWillUnmount() {
    this.handleClickThrottled.cancel();
  }

  render() {
    return <button onClick={this.handleClickThrottled}>Load more </button>;
  }

  handleClick() {
    this.props.loadMore();
  }
}
```

### Debounce

- Ensures a function won't be executed until after a certain amount of time has passed since it was last called
- Useful when one has to perform some expensive calculation in response to and event that might dispatch rapidly

```js
import debounce from "lodash.debounce";

class Searchbox extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.emitChangeDebounced = debounce(this.emitChange, 250);
  }

  componentWillUnmount() {
    this.emitChangeDebounced.cancel();
  }

  render() {
    return (
      <input
        type="text"
        onChange={this.handleChange}
        placeholder="Search..."
        defaultValue={this.props.value}
      />
    );
  }

  emitChange(value) {
    this.props.onChange(value);
  }
}
```

### requestAnimationFrame throttling

- requestAnimationFrame is a way of queuing a function to be executed in the browser at the optimal time for rendering performance
- A function that's queued with `requestAnimationFrame` will fire in the next frame
- Browser will work hard to ensure that there are 60 fps
- But if the browser is unstable, it will naturally limit the amount of frames in a second
- E.g. a device able to handle 30fps and so one will only get 30 frames in that second
- Using requestAnimationFrame for throttling is a useful technique in that it prevents one from doing more than 60 update in a second since if one does 100 updates in a second, it creates additional work for the browser that the user won't see anyways

```js
import rafSchedule from 'rad-schd';

class ScrollListener extends React.Component {
  constructor(props) {
    super(props);

    this.handleScroll = this.handleScroll.bind(this);

    this.scheduleUpdate = rafSchedule(
      point => this.props.onScroll(point)
    );
  }

  handleScroll(e) {
    this.scheduleUpdate({ x: e.clientX, y: clientY });
  }

  componentWillUnmount() {
    this.scheduleUpdate.cancel();
  }

  render() {
    return (
      <div onScroll={this.handleScroll}>
        <img src="/my-image..." />
      </div>
    );
  }
}
```

### Testing rate limiting

- Jest allows mock timer to fast forward time
- raf-stub can be useful for controlling the animation frames