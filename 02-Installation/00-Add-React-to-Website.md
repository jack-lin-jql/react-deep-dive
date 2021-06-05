# Add React in One Min

## Step 1: Add a DOM container to the HTML

- Open the HTML page and add an empty `<div>` tag to mark the spot where one wants to display something with React

```
<!-- ... existing HTML ... -->

<div id="like_button_container"></div>

<!-- ... existing HTML ... -->
```

- The unique `id` HTML attribute will allow JS to find the element later and display a React component inside of it
- A "container" `<div>` like this can be placed anywhere inside the `<body>` tag
- One can have as many independent DOM containers on a page as needed

## Step 2: Add the script tags

```
  <!-- ... other HTML ... -->

  <!-- Load React. -->
  <!-- Note: when deploying, replace "development.js" with "production.min.js". -->
  <script src="https://unpkg.com/react@17/umd/react.development.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js" crossorigin></script>

  <!-- Load our React component. -->
  <script src="like_button.js"></script>

</body>
```

- First two tags here loads React, the third loads the button component code

## Step 3: Create a React component

```
// Button component code

const domContainer = document.querySelector('#like_button_container');
ReactDOM.render(e(LikeButton), domContainer);

```

- Note that this component can be reused as follows

```
// In HTML
<p>
  This is the first comment.
  <!-- We will put our React component inside this div. -->
  <div class="like_button_container" data-commentid="1"></div>
</p>

<p>
  This is the second comment.
  <!-- We will put our React component inside this div. -->
  <div class="like_button_container" data-commentid="2"></div>
</p>

<p>
  This is the third comment.
  <!-- We will put our React component inside this div. -->
  <div class="like_button_container" data-commentid="3"></div>
</p>

// In component where DOM injection happens
// Find all DOM containers, and render Like buttons into them.
document.querySelectorAll('.like_button_container')
  .forEach(domContainer => {
    // Read the comment ID from a data-* attribute.
    const commentID = parseInt(domContainer.dataset.commentid, 10);
    ReactDOM.render(
      e(LikeButton, { commentID: commentID }),
      domContainer
    );
  });
```

# Try with JSX

- Above examples rely on features that are natively supported by the browsers. Hence the reason for using a JS function call to tell React what to display

```
const e = React.createElement;

// Display a "Like" <button>
return e(
  'button',
  { onClick: () => this.setState({ liked: true })},
  'Like
);
```

- In JSX:

```
return (
  <button onClick={() => this.setState({ liked: true })}>
    Like
  </button>
)
```

- The two snippets are equivalent

### Quickly try JSX

- The quickest way to try JSX is to add the following script tag

```
<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
```

- One can then use JSX in any `<script>` tag by adding `type="text/babel"` attribute to it
- Caution: this approach is fine for simple demos, but it make the website very slow and **isn't suitable for product**. Setup a JSX preprocessor to convert instead

# Add JSX to a project

- This is essentially a lot like adding a CSS preprocessor

```
npm init -y
npm install babel-cli@6 babel-preset-react-app@3
```

# Run JSX preprocessor

- Create a folder `src` and run the following
`npx babel --watch src --out-dir . --presets react-app/prod`
- Note that `npx` here it a package runner tool that comes with npm@5.2+
- The previous command starts an automated watcher for JSX
- Now, if a file is created in `src/like_button.js` containing JSX code, the watcher will create a preprocessed `like_button.js` with the plain JS code suitable for the browser. Upon editing the source file with JSX, the transform reruns automatically
- The added babel also transpiles newer JS syntax into older JS versions