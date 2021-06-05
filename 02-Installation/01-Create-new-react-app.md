# Recommended tool-chains by the React team

- Use [Create React App](https://github.com/facebook/create-react-app) to create a new SPA
- Use [Next.js](https://nextjs.org/) for server-rendered websites with Node.js
- Use [Gatsby](https://www.gatsbyjs.com) for static content-oriented website
- [Neutrino](https://neutrinojs.org/) is popular for building a component library or integrating with an existing codebase

## CRA

- A comfortable environment for learning React and the best way to start building a new SPA
- It sets the dev environment up so one can use the latest JS features, provides a nice dev experience, and optimizes one's app for production

```
// To create a project
npm create-react-app my-app
cd my-app
npm start
```

- CRA doesn't handle backend logic or databases, it simply creates a frontend build pipeline, so one can use it with any backend desired
- Under the hood, Babel and webpack are used but abstracted away
- Use `npm run build` to create an optimized build for production

## Next.js

- Lightweight framework for static and server-rendered applications built with React
- Includes styling and routing solutions out of the box, and **assumes one's using Node.js as the server environment**

## Gatsby

- Best way to create static websites with React
- Lets one use React components, but outputs pre-rendered HTML and CSS to guarantee the fastest load time

## More flexible tool-chains

- [Neutrino](https://neutrinojs.org/) combines the power of webpack with the simplicity of presets and includes a preset for React apps and React components
- [Nx](https://nx.dev/react) is a toolkit for full-stack monorepo development with built-in support for React, Next.js, Express and more
- [Parcel](https://parceljs.org/) is a fast, zero config web app bundler that works with React
- [Razzle](https://github.com/jaredpalmer/razzle) is a server-rendering framework without the need for config but offers more flexibility than Next.js

# Creating a tool-chain from scratch

- A JS build tool-chain typically consists of the following
1. A package manager such as yarn or npm which allows users to take advantage of a vast ecosystem of 3rd party packages
2. A bundler, such as webpack or Parcel that lets one write modular code and bundle into small packages to optimize load time
3. A compiler such as Babel which lets one write modern JS code that works in older browsers