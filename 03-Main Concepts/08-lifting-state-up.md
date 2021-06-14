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

## Adding a second input

- Now to add an input for Fahrenheit

```
// Extract a TemperatureInput from Calculator
const scaleNames = {
  c: 'Celsius',
  f: 'Fahrenheit',
};

class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = { temperature: '' };
  }

  handleChange(e) {
    this.setState({ temperature: e.target.value });
  }

  render() {
    const temperature = this.state.temperature;
    const scale = this.props.scale;

    return (
      <fieldset>
        <legend>Enter temperature in {scaleName[scale]}:</legend>
        <input value={temperature} onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```

```
// Updated Calculator
class Calculator extends React.Component {
  render() {
    return (
      <div>
        <TemperatureInput scale="c" />
        <TemperatureInput scale="f" />
      </div>
    );
  }
}
```

- Now, there're two inputs, but when on is entered, the other doesn't update which contradicts the calculation requirement, so they need to be synced up
- One also can't display the `BoilingVerdict` from `Calculator` since the calculator doesn't know the current temperature

## Writing conversion formula

```
function toCelsius(fahrenheit) {
  return (fahrenheit - 32) * 5 / 9;
}

function toFahrenheit(celsius) {
  return (celsius * 9 / 5) + 32;
}
```

```
// A converter function to return a string
function tryConvert(temperature, convert) {
  const input = parseFloat(temperature);
  if (Number.isNaN(input)) {
    return '';
  }

  const output = convert(input);
  const rounded = Math.round(output * 1000) / 1000;
  return rounded.toString();
}
```

## Lifting state up

- At the moment, both `TemperatureInput` components independently keep their values in their local state

```
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    // ...  
```

- Since we need these two inputs to be in sync, we need to life the state up
- In React, sharing state is accomplished by moving it up to the closest common ancestor of the components that need it - "lifting state up", so we'll remove the local state from `TemperatureInput` and move it into `Calculator` instead
- `Calculator` then becomes the source of truth, keeping the two inputs in sync

- Since props are read-only, `TemperatureInput` has no control over them. In React, this is usually solved by making a component "controlled". Just like the DOM `<input>` accepting both a `value` and an `onChange` prop, `TemperatureInput` can also accept both `temperature` and `onTemperatureChange` props from `Calculator`
- Note: there is no special meaning to the names, but naming as `value` and `onChange` is a common convention

```
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.props.onTemperatureChange(e.target.value);
  }

  render() {
    const temperature = this.props.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature} onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```

```
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
    this.state = { temperature: '', scale: 'c' };
  }

  handleCelsiusChange(temperature) {
    thi.setState({ temperature, scale: 'c' });
  }

  handleFahrenheitChange(temperature) {
    thi.setState({ temperature, scale: 'f' });
  }

  render() {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

    return (
      <div>
        <TemperatureInput
          scale="c"
          temperature={celsius}
          onTemperatureChange={this.handleCelsiusChange}
        />
        <TemperatureInput
          scale="f"
          temperature={fahrenheit}
          onTemperatureChange={this.handleFahrenheitChange}
        />
        <BoilingVerdict
          celsius={parseFloat(celsius)}
        />
      </div>
    );
  }
}
```

- So what's going on here?
1. React calls the function specified as `onChange` (e.g. handleChange) on the DOM `<input>` (e.g. TemperatureInput component)
2. `handleChange` in `TemperatureInput` calls `this.props.onTemperatureChange()` with the new value
3. Either `handleCelsiusChange` or `handleFahrenheitChange` method get called depending on which input was edited
4. `Calculator` component asks React to re-render itself through calling `this.setState()` with the new input value and the current scale of the input just edited
5. React calls `Calculator`'s `render` to learn the new UI appearance. C and F recomputation happens here
6. React then calls `render` of individual `TemperatureInput` components with new props specified by the `Calculator`
7. React calls `render` of BoilingVerdict` with the new prop values
8. React DOM updates the DOM with the boiling verdict to match the latest input values. Inputs also update accordingly

## Lessons learned

- There should be a single source of truth for any data changes in a React application
- Normally, states are first added to the component that needs it for rendering, and if other components also need it, one can lift up their state to the closest common ancestor instead of trying to sync the states between different components. Relying on the **top-down data flow**
- Lifting state involves with more boilerplate code than two-way binding approaches, but as a benefit, it takes less work to find and isolate bugs since the surface area for bugs get greatly reduced (state lives in some component and that component alone can change it)
- If something can be derived from either props or state, it probably shouldn't be in the state. E.g. instead of storing both `celsiusValue` and `fahrenheitValue`, one can just store the last edited `temperature` and `scale`. Allowing clear or apply rounding without losing any precision