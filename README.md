
# Exchange Server Side Rendering

A brief description of what this project perform Server Side Rendering


## bable setup


```npm i webpack webpack-cli webpack-node-externals @babel/core babel-loader @babel/preset-env @babel/preset-react fs path npm-run-all nodemon express ```
## Step 1: Creating the React App and Modifying the App Component

Let’s call the app, react-ssr-example:

```npx create-react-app react-ssr-example```

Open the App.js file in the src directory

```

import React from 'react';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        {/* <img src={logo} className="App-logo" alt="logo" /> */}
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>      
        <h1>       
          WELCOME HERE
        </h1>
    
      </header>
    </div>
  );
}

export default App;

```

Let’s open the index.js file in the src directory:


```
ReactDOM.render( <React.StrictMode> <App /> </React.StrictMode>, document.getElementById('root'));

replace with 
 
ReactDOM.hydrate( <React.StrictMode> <App /> </React.StrictMode>, document.getElementById('root'));
 ```


```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App.js';
import reportWebVitals from './reportWebVitals.js';

// const root = ReactDOM.createRoot(document.getElementById('root'));
ReactDOM.hydrate(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

```

## Step 2 — Creating an Express Server and Rendering the App Component

create a new server folder in the project’s root directory:

inside of the server folder, create a new index.js file that will contain the Express server code:


```
import path from 'path';
import fs from 'fs';

import React from 'react';
import ReactDOMServer from 'react-dom/server';
import express from 'express';

import App from '../src/App.js';
const PORT =  3006;
const app = express();

app.get('/', (req, res) => {

    const app = ReactDOMServer.renderToString(<App />);
    const indexFile = path.resolve('./build/index.html');
  
    fs.readFile(indexFile, 'utf8', (err, data) => {
      if (err) {
        console.error('Something went wrong:', err);
        return res.status(500).send('Oops, better luck next time!');
      }
  
      return res.send(
        data.replace('<div id="root"></div>', `<div id="root">${app}</div>`)
      );
    });
  });
  
  // app.use(express.static('./build'));
  
  app.listen(PORT, () => {
    console.log(`Server is listening on port ${PORT}`);
  });
```

-  Express is used to serve contents from the build directory as static files.
* ReactDOMServer’s renderToString is used to render the app to a static HTML string.
+ The static index.html file from the built client app is read. The app’s static content is injected into the <div> with an id of "root". This is sent as a response to the request.
## Step 3 — Configuring webpack, Babel, and npm Scripts

```
To avoid  conflicts, includes this installation step.
webpack, webpack-cli, webpack-node-externals, @babel/core, babel-loader, @babel/preset-env, @babel/preset-react
```


` Create a weback.server.js file to root directory`

```
import path from "path"
import nodeExternals from 'webpack-node-externals'

export const serverConfig = {
  entry: './server/index.js',
  target: 'node',
  externals: [nodeExternals()],
  output: {
    path: path.resolve('server-build'),
    filename: 'index.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader'
      }
    ]
  }
};
```

Next, create a new Babel configuration file in the project’s root directory:

`Create babelrc.json file to root directory`
```
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react"
  ]
}
```

With this configuration, the transpiled server bundle will be output to the server-build folder in a file called index.js.
## Step 4 : Update package.json

Now, visit package.json

```
Note: You do not need to modify the existing "start", "build", "test", and "eject" scripts in the package.json file.
```

```
  "scripts": {
    
    "dev:build-server": "NODE_ENV=development webpack --config webpack.server.js    --mode=development -w",
    "dev:start": "nodemon ./server-build/index.js",
    "dev": "npm-run-all --parallel build dev:*"
  },
```
```
npm install nodemon --save-dev
npm install npm-run-all --save-dev
```

```
npm run dev
```
