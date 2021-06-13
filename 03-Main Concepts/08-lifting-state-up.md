# Lifting state up

- Often, several components need to reflect the same changing data. It is recommended to lift the shared state up to their closest common ancestor in this situation
- Let's create a temperature calculator for example

```
function BoilingVerdict(props) {
  if (props.celsius >= 100) {
    return <p>The water would boil.</p>;
  }

  return <p>The water would not boil</p>;
}
```

```
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = { temperature: '' };
  }

  handleChange(e) {
    this.setState({ temperature: e.target.value });
  }

  render() {
    const temperature = this.state.temperature;;
    return (
      <fieldset>
        <legend>Enter temperature in Celsius:</legend>
        <input
          value={temperature}
          onChange={this.handleChange}
        />
        <BoilingVerdict celsius{parseFloat(temperature)} />
      </fieldset>
    );
  }
}
```

