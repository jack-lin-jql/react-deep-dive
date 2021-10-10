# Styling and CSS

## How does one add CSS classes to components?

```js
render() {
  return <span className="menu navigation-menu">Menu</span>;
}
```

```js
render() {
  let className='menu';

  if (this.props.isActive) {
    className += ' menu-active';
  }

  return <span className={className}>Menu</span>;
}
```

## Are inline-styles bad?

- CSS classes are generally better for performance than inline styles

## What is CSS-in-JS?

- "CSS-in-JS" refers the the pattern where CSS is composed using JS instead of defined in external files
- This functionality isn't a part of React, but provided by 3rd party libraries
- React doesn't have an opinion about how styles are defined