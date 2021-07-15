# Reconciliation

- React provides a declarative API so that one doesn't have to worry about exactly what changes on every update
- This makes writing applications a lot easier, but it might not be obvious how this is implemented within React
- This document helps with explaining the choices made in React's diffing algorithm so that component updates are predictable while being fast enough for high-performance apps

## Motivation

- When using React, at a single point in time, one can think of the `render()` function as creating a tree of React elements
- On the next state or props update, that `render()` function will return a different tree of React elements
- React then figures out how to efficiently update the UI to match the most recent tree
- There are some generic solutions to this algorithmic problem of generating the minimum number of operations to transform one tree into another. However, the state of the art algorithms have a complexity in the order of O(n^3) where n is the number of elements in the tree (oh damn, that's quite expensive)
- If we used this in React, displaying 1000 elements would require in the order of one billion comparisons, way too expensive
- Instead, **React implements a heuristic O(n) algorithm based on two assumptions**
  1. Two elements of different types will produce different trees
  2. The develop can hint at which child elements may be stable across different renders with a `key` prop
- In practice, these assumptions are valid for almost all practical use cases

## The diffing algorithm

- When diffing two trees, React first compares the two root elements
- The behavior is different depending on the types of the root elements

### Elements of different types

- When the root element have different types, React will tear down the old tree and build the new tree from scratch
- Going from `<a>` to `<img>`, or from `<Article>` to `<Comment>` will lead to a full rebuild
- When tearing down a tree, old DOM nodes are destroyed. Component instances receive `componentWillUnmount()`
- When building up a new tree, new DOM nodes are inserted into the DOM. Component instances receive `UNSAFE_componentWillMount()` and then `componentDidMount()`
- Any state associated with the old tree is lost
- Any components below the root will also get unmounted and have their state destroyed. E.g., when diffing:

```
<div>
  <Counter />
</div>

<span>
  <Counter />
<span>
```

- This will destroy the old `Counter` and remount a new one
- Note: These methods are considered legacy and one should avoid them in new code:
  - `UNSAFE_componentWillMount()`

### DOM elements of the same type

- When comparing two React DOM elements of the same type, React looks at the attributes of both, keeps the same underlying DOM node, and only updates the changed attributes. E.g.:

```
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

- By comparing these two elements, React knows to only modify the `className` on the underlying DOM node
- When updating `style`, React also knows to update only the properties that changed. E.g.:

```
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

- When converting between these two elements, React knows to only modify the `color` style, not the `fontWeight`
- After handling the DOM node, React then recurses on the children

### Component elements of the same type

- When a component updates, the instance stays the same, so that state is maintained across renders
- React updates the props of the underlying component instance to match the new element, and calls `UNSAFE_componentWillReceiveProps()`, `UNSAFE_componentWillUpdate()`, and `componentDidUpdate()` on the underlying instance
- Next, the `render()` method is called and the diff algorithm recurses on the previous result and the new result
- Note: These methods are considered legacy and one should avoid them in new code:
  - `UNSAFE_componentWillReceiveProps()`
  - `UNSAFE_componentWillUpdate()`

### Recursing on children

- By default, when recursing on the children of a DOM node, React just iterates over both lists of children at the same time and generates a mutation whenever there's difference
- E.g., when adding an element at the end of the children, converting between these two trees works well:

```
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

- React will match the two `<li>first</li>` trees, similarly with the `<li>second</li>` trees, and then insert the `<li>third</li>` tree
- If one implements it naively, inserting an element at the beginning has worse performance. E.g., converting between two trees works poorly:

```
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

- Here, React will mutate every child instead of realizing it can keep `Duke` and `Villanova` subtrees intact. This inefficiency can be a problem

### Keys

- In order to solve this issue, React supports a `key` attribute
- When children have keys, React uses the key to match children in the original tree with children in the subsequent tree
- E.g. adding a `key` to the previously inefficient example can make the tree conversion more efficient

```
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

- Now React knows that the element with key `'2014'` is the new one, and the elements with the keys `'2015'` and `'2016'` have just moved
- In practice, finding a key is usually not hard. The element one is going to display may already have a unique ID, so the key can just come from the data:

`<li key={item.id}>{item.name}</li>`

- When that's not the case, one can add a new ID property to the model or hash some parts of the content to generate a key
- The key only has to be unique among its siblings, not globally unique
- As a last resort, one can pass an item's index in the array as a key. This can work well if the items are never reordered, but reorders will be slow
- Reorders can also cause issues with component state when indexes are used as keys
- Component instances are updated and reused on their key. If the key is an index, moving an item changes it. As a result, a component state for things like uncontrolled inputs can get mixed up and updated in unexpected ways
- Go to official document for an real example

## Tradeoffs

- It's important to remember that the reconciliation algorithm is an implementation detail
- React could rerender the whole app on every action; the end result would be the same
- Just to be clear, rerender in this context means calling `render` for all components, it doesn't mean React will unmount and remount them!
- It will only apply the differences following the rules stated in the previous sections
- In the current implementation, one can express the fact that a subtree has been moved amongst its siblings, but one cannot tell that it has moved somewhere else. The algorithm will rerender the full subtree
- Since React relies on heuristics, if the assumptions behind them aren't met, performances will suffer
  1. The algorithm won't try to match subtrees of different component types. If one sees themselves alternating between two component types with very similar output, one may want to make it the same type. This in practice hasn't been found to be an issue
  2. Keys should be stable, predictable, and unique. Unstable keys (like those produced by `Math.random()`) will cause many component instances and DOM nodes to be unnecessarily recreated, which can cause performance degradation and lost state in child components