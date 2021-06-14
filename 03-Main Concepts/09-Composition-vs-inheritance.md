# Composition vs inheritance

- React has a powerful composition model so it's recommended in using composition instead of inheritance to reuse code between components

## Containment

- Some components don't know their children ahead of time, especially common for components like `Sidebar` or `Dialog` that represent generic boxes
- It's recommended that such components use `children` prop to pass children elements directly to their output

```
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

```
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

- Anything inside the `<FancyBorder>` JSX tag gets passed into the `FancyBorder` component as a children prop. Since `FancyBorder` renders `{props.children}` inside a `<div>`, the passed elements appear in the final output