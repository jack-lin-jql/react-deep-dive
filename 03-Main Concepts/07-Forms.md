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

