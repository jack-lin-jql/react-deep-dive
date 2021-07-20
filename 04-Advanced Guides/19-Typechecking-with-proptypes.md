# Typechecking with PropTypes

- React PropTypes has been moved to the package [prop-types](https://www.npmjs.com/package/prop-types) since v15.5

- As an app grows, one can catch a lot of bugs with typechecking
- For some applications, one can use JS extensions like Flow or TS to typechecking the entire app, but even if one doesn't use those, React has some built-in typechecking abilities
- To run typechecking on props for a component, one can assign the special `propTypes` property

```
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
```

- In the previous example, we're using a class component, but the same functionality could also be applied to function components, or components created by `React.memo` or `React.forwardRef`
- `PropTypes` export a range of validators that can be used to make sure the data one receive is valid
- In the example, `PropTypes.string` was used. When an invalid value is provided for a prop, a warning will be shown in the JS console
- For performance reasons, `propTypes` is only checked in development mode

## PropTypes

- Here's an example documenting different validators provided:

```
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // One can declare that a prop is a specific JS type. By default, these are all optional
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // Anything that can be rendered: numbers, strings, elements or an array (or fragment) containing these types
  optionalNode: PropTypes.node,

  // A React element
  optionalElement: PropTypes.element,

  // A React element type (ie. MyComponent)
  optionalElementType: PropTypes.elementType,

  // One can also declare that a prop is an instance of a class. This uses JS's instanceof operator 
  optionalMessage: PropTypes.instanceOf(Message),

  // One can ensure that a prop is limited to specific values by treating it as an enum
  optionalEnum: PropTypes.oneOf(['New', 'Photos']),

  // An object that could be one of many types
  optionalUnion: PropTypes.oneOfTypes([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // An array of a certain type
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // An object with property values of a certain type
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // An object taking on a particular shape
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // An object with warnings on extra properties
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number
  }),

  // One can chain any of the above with `isRequired` to make sure a warning is shown if the prop isn't provided
  requiredFunc: PropTypes.func.isRequired,

  // A required value of any data type
  requiredAny: PropTypes.any.isRequired,

  // One can also specify a custom validator, it should return an Error object if the validation fails. Don't `console.warn` or throw, as this won't work inside `oneOfType`
  customProp: function(props, propName, componentName) {
    if(!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' + ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // One can also supply a custom validator to `arrayOf` and `objectOf`.
  // It should return an Error object if the validation fails. The validator will be called for each key in the array or object. The first two arguments of the validator are the array or object itself, and the current item's key
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' + ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

## Requiring single child

- With `PropTypes.element` one can specify that only a single child can be passed to a component as children

```
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // This must be exactly one element or it'll warn.
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children.PropTypes.element.isRequired
};
```

## Default prop values

- One can define default values for their props by assigning to the special `defaultProps` property:

```
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// Specifies the default value for props:
Greeting.defaultProps = {
  name: 'Stranger'
};

// Renders "Hello, Stranger":
ReactDOM.render(
  <Greeting />,
  document.getElementById('example')
);
```

- If one is using a Babel transform like `transform-class-properties`, one can also declare `defaultProps` as static property within a React component class
- This syntax hasn't yet been finalized though and will require a compilation step to work within a browser

```
class Greeting extends React.Component {
  static defaultProps = {
    name: 'stranger'
  }

  render() {
    return (
      <div>Hello, {this.props.name}</div>
    );
  }
}
```

- The `defaultProps` will be used to ensure that `this.props.name` will have a value if it was not specified by the parent component
- The `propTypes` typechecking happens after `defaultProps` are resolved, so typechecking will also apply to the `defaultProps`

## Function components

- If one is using function components in their regular development, one may want to make some small changes to allow PropTypes to be properly applied
- Say a component look like:

```
export default function HelloWorldComponent({ name }) {
  return (
    <div>Hello, {name}</div>
  );
}
```

- To add PropTypes, one may want to declare the component in a separate function before exporting, like this:

```
function HelloWorldComponent({ name }) {
  return (
    <div>Hello, {name}</div>
  );
}

export default HellWorldComponent;
```

- Then one can add PropTypes directly to the component: 

```
import PropTypes from 'prop-types';

function HelloWorldComponent({ name }) {
  return (
    <div>Hello, {name}</div>
  );
}

HelloWorldComponent.propTypes = {
  name: PropTypes.string
};

export default HellWorldComponent;
```