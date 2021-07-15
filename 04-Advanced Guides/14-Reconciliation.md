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