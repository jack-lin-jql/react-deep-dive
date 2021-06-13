# Forms 

- HTML form elements work a bit different from other DOM elements in React, because form elements naturally keep some internal state. E.g. this plain HTML accepts a single name

```
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```

- This form has the default HTML form behavior of browsing to a new page when the user submits the form
- If one wants this behavior in React, it just works. But in most cases, it's convenient to have a JS function that handles the submission of the form and has access to the data that the user entered into the form
- The standard way of achieving this is with a technique called "controlled components"

## Controlled components

- In HTML, form elements such as `<input>`, `<textarea>`, and `<select>` typically maintain their own state and update it based on user input
- In React, mutable state is typically kept in the state property of components and only updated with `setState()`
- One can combine the two by making the React state be the "single source of truth"
- The React component that renders the form would then also control what happens in that form on subsequent user input
- An input form element whose value is controlled by React in this way is called a "controlled component"

```
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChang.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return(
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

- Since the `value` attribute is set on the form element, the displayed value will always be `this.state.value`, making the React state the source of truth
- Since `handleChange` runs on every keystroke to update the React state, the displayed value will update as the user types
- With a controlled component, the input's value is always driven by React state
- This may mean a bit more code, but one can now pass the value to other UI elements too, or reset it from other event handlers

## The textarea tag

- In HTML, a `<textarea>` element defines its text by its children

```
<textarea>
  Hello there, this is some text in a text area
</textarea>
```

- In React, a `<textarea>` uses a `value` attribute instead, so this way, a form using a `<textarea>` can be written very similarly to a form that uses a single-line input

```
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 'Please write an essay about your favorite DOM element.'
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit(event) {
    alert('An essay was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Essay:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" >
      </form>
    )
  }
}
// Note here that `this.state.value` has been initialized with some default string in the constructor and therefore the corresponding text area starts with the same string in it
```

## The select tag

- In HTML, `<select>` creates a drop-down list

```
<select>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option selected value="coconut">Coconut</option>
  <option value="mango">Mango</option>
</select>
```

- Note here that the coconut option is initially selected due to the `selected` attribute
- In React, instead of using the `selected` attribute, we use a `value` attribute on the root `select` tag
- This is more convenient in a controlled component since one only need to update it in one place

```
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'coconut' };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit(event) {
    alert('Your favorite flavor is: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Pick your favorite flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Submit />
      </form>
    );
  }
}
```

- This makes so that `<input type="text">`, `<textarea>`, and `<select>` all work very similarly where they all accept a `value` attribute that one can use to implement a controlled component
- NOTE: one can pass an array into the value attribute, allowing the selection of multiple options in a select tag

`<select multiple={true} value={['B', 'C']}>`

## The file input tag

- In HTML, `<input type="file">` lets the user choose one or more files from their device storage to be uploaded to a server or manipulated by JS via the File API

`<input type="file" />`

- Since its value is read-only, it's an **uncontrolled** component in React

## Handling multiple inputs

- When one needs to handle multiple controlled `input` elements, one can add a `name` attribute to each element and let the handler function choose what to do based on the value of the `event.target.name`

```
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange}
          />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange}
          />
        </label>
      </form>
    );
  }
}
```

- Note how we just used ES6's computed property name to update the state key corresponding to the given input name

```
this.setState({
  [name]: value
});
```

```
// Equivalent to this ES5 code
var partialState = {};
partialState[name] = value;
this.setState(partialState);
```

- Also note that since `setState()` auto merges a partial state into the current state, one only needs to call it with the changed parts

## Controlled input null value

- Specifying the value prop on a controlled component prevents the user from changing the input unless one desire so
- If one's specified a `value` but the input is still editable, one may have accidentally set `value` to `undefined` or `null`

```
// Input here is locked at first and then becomes editable after a short delay
ReactDOM.render(<input value="hi" />, mountNode);

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode);
}, 1000);
```

## Alternatives to controlled components

- Controlled components can sometimes be tedious since one needs to write an event handler for every way data can change and pipe all of the input state through a React component
- It becomes particularly annoying when one is converting a preexisting codebase to React or integrating a React application with a non-React library
- Uncontrolled components might be your friend here

## Fully-fledged solutions

- If one is looking for a complete solution including validation, keeping track of the visited fields and handling form submission, [Formik](https://formik.org/) is one of the popular choices
- However, it is built on the same principles of controlled components and managing state