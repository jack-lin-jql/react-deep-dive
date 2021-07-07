# Optimizing performance

- Internally, React uses several clever techniques to minimize the number of costly DOM operations required to update the UI
- For many applications, using React will lead to a fast UI without doing much work to specifically optimize for performance
- Nevertheless, there are several ways one can speed up a React app

## Use the production build

- If one is benchmarking or experiencing performance problems in a React app, makes ure to test with the minified production build
- By default, React includes many helpful warnings, which are very useful in development, but they make React larger and slower so one should make sure to use the production version when deploying the app
- If you aren't sure whether a build process is set up correctly, check it by installing the React devtool for Chrome. If a site with React is in production mode, the icon will have a dark background

![](./imgs/09-production-devtool-icon.png)

- When visiting in development mode, the icon will have a red background

![](./imgs/09-development-devtool-icon.png)

- It's expected that one uses the development mode when working on the app and production mode when deploying app to the users

### CRA

- If a project is built with CRA, simply run `npm run build`
- This creates a production build of the app in the `build/` folder of the project
- This is only necessary before deploying to production, for normal development, use `npm start`

### Single-file builds

- React offers production-ready version of React and React DOM as single files

```
<script src="https://unpkg.com/react@17/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom@17/umd/react-dom.production.min.js"></script>
```

- Remember that only React files ending with `.production.min.js` are suitable for production

### Brunch

- Brunch is a build tool like Webpack but with a different flavor
- For the most efficient Brunch production build, install [terser-brunch](https://github.com/brunch/terser-brunch)

```
# If you use npm
npm install --save-dev terser-brunch

# If you use Yarn
yarn add --dev terser-brunch
```
- To create a production build, add the `-p` flag to the `build` command

`brunch build -p`

- Remember that one only needs to do this for production builds. One shouldn't pass the `-p` flag or apply this plugin in development, because it'll hide useful React warnings and make the build much slower

### Browserify

- For the most efficient Browserify production build, install a few plugins

```
# If you use npm
npm install --save-dev envify terser uglifyify

# If you use Yarn
yarn add --dev envify terser uglifyify
```

- To create a production build, make sure to add these transforms (**order matters**)
  - `envify` transform ensures the right build environment is set. Make it global (`-g`)
  - `uglifyify` transform removes development imports. Make it global too (`-g`)
  - Resulting bundle is piped to `terser` for mangling

```
// E.g.
browserify ./index.js \
  -g [ envify --NODE_ENV production ] \
  -g uglifyify \
  | terser --compress --mangle > ./bundle.js
```

- Remember that one only needs to do this for production builds

### Rollup

- For the most efficient Rollup production build

```
# If you use npm
npm install --save-dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-terser

# If you use Yarn
yarn add --dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-terser
```

- To create a production build, make sure to add these transforms (**order matters**)
  - `replace` plugin ensures the right build env is set
  - `commonjs` provides support for CommonJS in Rollup
  - `terser` compresses and mangles to the final bundle

```
plugins: [
  // ...
  require('rollup-plugin-replace')({
    'process.env.NODE_ENV': JSON.stringify('production')
  }),
  require('rollup-plugin-commonjs')(),
  require('rollup-plugin-terser')(),
  // ...
]
```

### webpack

- Note: if using CRA, webpack is already used, so this is only applicable to configuring webpack directly
- Webpack v4+ will minify one's code by default in production mode

```
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  mode: 'production',
  optimization: {
    minimizer: [new TerserPlugin({ /* additional options here */ })],
  },
};
```

## Profiling components with the devtool profiler

- `react-dom` 16.5+ and `react-native` 0.57+ provide enhanced profiling capabilities in DEV mode with the React DevTools Profiler
- 