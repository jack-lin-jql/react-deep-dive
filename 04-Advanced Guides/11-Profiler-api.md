# Profiler API

- The `Profiler` measures how often a React application renders and what the "cost" of rendering is
- It's purpose is to help identify parts of an application that are slow and may benefit from optimization such as memoization
- Note: Profiling adds some additional overhead, so it's disabled in the production build
- Note: To opt into production profiling, React provides a special production build with profiling enabled

## Usage

- A `Profiler` can be added anywhere in a React tree to measure the cost of rendering that part of the tree
- It requires 2 props
  - An `id` (string)
  - An `onRender` callback function which React calls anytime a component within the tree "commits" an update
- For example, to profile `Navigation` component and its descendants:

```
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Main {...props} />
  </App>
);
```

- Multiple `Profiler` components can be used to measure different parts of an application

```
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Profiler id="Main" onRender={callback}>
      <Main {...props} />
    </Profiler>
  </App>
);
```

- `Profiler` components can also be nested to measure different components within the same subtree

```
render(
  <App>
    <Profiler id="Panel" onRender={callback}>
      <Panel {...props}>
        <Profiler id="Content" onRender={callback}>
          <Content {...props} />
        </Profiler>
        <Profiler id="PreviewPane" onRender={callback}>
          <PreviewPane {...props} />
        </Profiler>
      </Panel>
    </Profiler>
  </App>
);
```

- Note: Although `Profiler` is a light-weight component, it should be used only when necessary; each use adds some CPU and memory overhead to an application

## onRender callback

- The `Profiler` requires an `onRender` function as a prop
- React calls this function any time a component within the profiled tree "commits" an update
- It receives parameters describing what was rendered and how long it took

```
function onRenderCallback(
  id, // The "id" prop of the Profiler tree that had just been committed
  phase, // Either "mount" (is tree had just been mounted) or "updated" (if re-rendered)
  actualDuration, // Time spend rendering the committed update
  baseDuration, // Estimated time to render the entire subtree without memoization
  startTime, // When React began rendering this update
  commitTime, // When React committed this update
  interactions // The Set of interactions belonging to this update
) {
  // Aggregate or log render timings...
}
```

- Closer look at each prop:
  - id: string - Can be used to identify which part of the tree was committed if one is using multiple Profilers
  - phase: "mount" | "update" - IDs whether the tree had just been mounted for the first time or re-rendered due to a change in props, state, or hooks
  - actualDuration: number - Time spent rendering the `Profiler` and its descendant for the current update. Indicates how well the subtree makes use of memoization. Ideally, this value should decrease significantly after the initial mount as many of the descendants wil only need to re-render if their specific props change
  - baseDuration: number - Duration of the most recent `render` time for each individual component within the `Profiler` tree. This value estimates worst-case cost of rendering (e.g. the initial mount or a tree with no memoization)
  - startTime: number - Timestamp when React began rendering the current update
  - commitTime: number - Timestamp when React committed the current update. This value is shared between all profilers in a commit, enabling them to be grouped if desirable
  - interactions: Set - Set of interactions that were being traced when the update was scheduled (e.g. when render or setState were called)
- Note: Interactions can be used to identify the cause of an update, although the API for tracing them is still experimental
