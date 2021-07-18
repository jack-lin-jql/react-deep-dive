# Static type checking

- Static type checkers type `Flow` and `TypeScript` identify certain types of problems before one even runs their code
- They can also improve developer workflow by adding features like auto-completion
- For this reason, it's recommended to use Flow or TS instead of `PropTypes` for larger code bases

## Flow

- Flow is a static type checker for JS code
- It's developed at Facebook and is often used with React
- It allows one to annotate the variables, functions and React components with a special type syntax, and catch mistakes early
- To use Flow:
  - Add Flow to the project as a dependency
  - Ensure that Flow syntax is stripped from the compiled code
  - Add type annotations and run Flow to check them

### Adding flow to a project

- Install with:

`yarn add --dev flow-bin`

- Or

`npm install flow-bin --save-dev`

- Add `flow` to the `"scripts"` section of `package.json`

```
{
  // ...
  "scripts": {
    "flow": "flow",
    // ...
  }
}
```

- Running

`yarn run flow init`

- Or

`npm run flow init`

- This will create a Flow config file that one will need to commit

### Stripping Flow syntax from the compiled code

- Flow extends the JS language with a special syntax for type annotations
- However, browsers aren't aware of this syntax, so one needs to make sure it doesn't end up in the compiled JS bundle that's sent to the browser
- To achieve this, it'll depend on the tools used to compile JS

#### CRA

- If a project was set up using CRA, Flow annotations are already being stripped by default, so one doesn't need to do anything else

#### Babel

- Note: The following aren't for CRA users even though CRA uses Babel under the hood
- If one manually configured Babel, one will need to install a special preset for Flow

`yarn add --dev @babel/preset-flow`

`npm install --save-dev @babel/preset-flow`

- Add the `flow` preset to the Babel config in `.babelrc`

```
{
  "presets": [
    "@babel/preset-flow",
    "react"
  ]
}
```

- This will let one use the Flow syntax in code
- Note: Flow doesn't require the `react` preset, but they're usually used together. Flow itself understands JSX out of the box

#### Other build steps

- If one doesn't use CRA or Babel, use `flow-remove-types` to strip the annotations

### Running Flow

`yarn flow`

`npm run flow`

### Adding Flow type annotations

- By default, Flow only checks the files that include this annotation:

`// @flow`

- Typically, it's placed at the top of a file, trying adding it to some file in the project and run `yarn flow` or `npm run flow` to see if any issues found
- There's an option to force Flow to check all files regardless of the annotation
- This can be too noisy for existing projects, but is reasonable for a new project if one wants to fully type it with Flow

## TypeScript

- TS is a **programming language** developed by Microsoft
- It's a typed superset of JS and includes its own compiler
- Being a typed language, TS can catch errors and bugs at build time, long before an app goes live
- To use TS:
  - Add TS as a dependency to project
  - Configure TS compiler options
  - Use the right file extensions
  - Add definitions for libraries one uses

### Using TS with CRA

- CRA supports TS out of the box
- To create a new project with TS support:

`npx create-react-app my-app --template typescript`

- One can also add it to an existing CRA project
- The following is a guide for manual setup which doesn't apply to CRA users

### Adding TS to a project

`yarn add --dev typescript`

`npm install typescript --save-dev`

- Installing TS gives access to the `tsc` command
- Before configuration, add `tsc` to scripts section of `package.json`

```
{
  // ...
  "scripts": {
    "build": "tsc,
    // ...
  },
  // ...
}
```

### Configuring TS compiler

- The compiler is no help until one tells it what to do
- In TS, these rules are defined in a special file called `tsconfig.json`
- To generate this file, run:

`yarn run tsc --init`

`npm tsc --init`

- Looking at the now generate `tsconfig.json`, one can see that there are many options to configure the compiler
- Of many options, let's look at `rootDir` and `outDir`
- In its true fashion, the compiler will take in TS files and generate JS files, however, we don't want to get confused with our source files and generated output
- Let's address this in two steps:
  - Let's arrange the project structure like the following
  
  ```
  -> package.json
  -> src
    -> index.ts
  -> tsconfig.json
  ```

  - Tell the compiler where the source code is and where the output should go
  
  ```
  // tsconfig.json
  {
    "compilerOptions": {
      // ...
      "rootDir": "src",
      "outDir": "build"
      // ...
    },
  }
  ```

- Now when the build script is run, the compiler will output the generated JS to the `build` folder
- [TS React Started](https://github.com/Microsoft/TypeScript-React-Starter/blob/master/tsconfig.json) provides a good set of rules to gets started in tsconfig.json
- Generally, one shouldn't keep the generated JS in source control, so add that build folder to `.gitignore`

### File extension

- In React, one most likely write components in a `.js` file. In TS, there're 2 file extensions:
  - `.ts` is the default file extension while `.tsx` is a special extension used for files which contain `JSX`

### Running TS

`yarn build`

`npm run build`

- If no output, it means that it completed successfully

### Type definitions

- To be able to show errors and hints from other packages, the compiler relies on declaration files
- A declaration file provides all the type information about a library
- This enables us to use JS libraries like those on npm in our project
- There're two main ways to get declarations for a library: 
  - Bundled
    - The library bundles its own declaration file, this is great for consumers since then all they need to do is install the library and use it right away
    - To check if a library has bundled types, look for an `index.d.ts` file in the project
    - Some libraries will have it specified in their `package.json` under the `typings` or `types` filed
  - [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)
    - This is a huge repository of declarations for libraries that don't bundle a declaration file
    - The declarations are crowd-sourced and managed by Microsoft and open source contributors
    - React for example doesn't bundle its own declaration files. Instead, one can get it from DefinitelyTyped 
    
    ```
    yarn add --dev @types/react

    npm i --save-dev @types/react
    ```

  - Local declarations
    - Sometimes the package that one uses doesn't bundle declarations nor is it available on DefinitelyTyped
    - In that case, one can have a local declaration file
    - Create a `declarations.d.ts` file in the root of a source directory

    ```
    // E.g.
    declare module 'querystring' {
      export function stringify(val: object): string
      export function parse(val: string): object
    }
    ```

## ReScript

- A typed language that compiles to JS, some of its core features are guaranteed 100% type coverage, first-class JSX support and dedicated React bindings to allow integration in existing JS/TS React codebases

## Kotlin

- A statically typed language by JetBrains
- Its target platforms include the JVM, Android, LLVM, and JS
- JetBrains develops and maintains several tools specifically for the React community: React bindings as well as Create React Kotlin App (which helps with building React apps with Kotlin with no build configuration)