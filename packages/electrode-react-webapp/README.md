# Electrode React Webapp

[![NPM version][npm-image]][npm-url] [![Dependency Status][daviddm-image]][daviddm-url] [![devDependency Status][daviddm-dev-image]][daviddm-dev-url] [![npm downloads][npm-downloads-image]][npm-downloads-url]

This is a Hapi plugin that register a default route for your Webapp to return
a bootstrapping React application.  With support for webpack dev server integrations.

## Installing

```bash
$ npm install electrode-react-webapp --save
```

## Usage

### Registering with Hapi

You can use this plugin by registering it with your Hapi server.

```js
const reactWebapp = server.register({
  register: require("electrode-react-webapp"),
  options: {
    pageTitle: "My Awesome React WebApp",
    paths: {
      "/{args*}": {
        view: "index",
        content: "<h1>Hello React!</h1>"
    }
  }
});
```

### Registering with electrode-server

To use this with electrode-server, add to the config you pass to `electrode-server`:

```js
const config = {
  plugins: {
    "electrode-react-webapp": {
      options: {
        pageTitle: "My Awesome React WebApp",
        paths: {
          "/{args*}": {
            view: "index",
            content: "<h1>Hello React!</h1>"
        },
        unbundledJS: {
          enterHead: [
            {src: "http://cdn.com/js/lib.js"}
          ]
        }
      }
    }
  }
}

require("electrode-server")(config);
```

## Default options

This plugin has some default options but you can override them by setting your own value.

The current defaults are:

```js
{
  pageTitle: "Untitled Electrode Web Application",
  webpackDev: process.env.WEBPACK_DEV === "true",
  renderJS: true,
  serverSideRendering: true,
  htmlFile: "node_modules/electrode-react-webapp/lib/index.html",
  devServer: {
    host: "127.0.0.1",
    port: "2992"
  },
  paths: {},
  stats: "dist/server/stats.json"
}
```

## Options

What you can do with the options:

-   `pageTitle` `(String)` The value to be shown in the browser's title bar
-   `webpackDev` `(Boolean)` whether to use webpack-dev-server's URLs for retrieving CSS and JS bundles.
-   `serverSideRendering` `(Boolean)` Toggle server-side rendering.
-   `htmlFile` `(String)` Absolute or relative path to the application root html file.
     It must contains the following placeholders:
    -   `{{PAGE_TITLE}}` page title.
    -   `{{WEBAPP_BUNDLES}}` injected `<script>` and `<link>` tags to load bundled JavaScript and Css
    -   `{{PREFETCH_BUNDLES}}` `<script>` tag containing code that will contains prefetched JavaScript code
    -   `{{SSR_CONTENT}}` injected content rendered on server side
-   `paths` `(Object)` An object of key/value pairs specifying paths within your application with their view and (optionally) initial content for server-side render
    -   _path_ `(Object)`
        -   `view` `(String)` Name of the view to be used for this path **required**
        -   `content` Content to be rendered by the server when server-side rendering is used _optional_ [see details](#content-details)
-   `unbundledJS` (Object) Specify JavaScript files to be loaded at an available extension point in the index template
    -   `enterHead` (Array) Array of script objects (`{ src: "path to file" }`) to be inserted as `<script>` tags in the document `head` before anything else. To load scripts asynchronously use `{ src: "...", async: true }` or `{ src: "...", defer: true }`
    -   `preBundle` (Array) Array of script objects (`{ src: "path to file" }`) to be inserted as `<script>` tags in the document `body` before the application's bundled JavaScript
-   `devServer` `(Object)` Options for webpack's DevServer
    -   `host` `(String)` The host that webpack-dev-server runs on
    -   `port` `(String)` The port that webpack-dev-server runs on
-   `prodBundleBase` `(String)` Base path to locate the JavaScript, CSS and manifest bundles. Defaults to "/js/". Should end with "/".

### Content details

The content you specify for your path is the entry to your React application when doing Server Side Rendering. 

It can be a string, a function, or an object.

#### `string`

If it's a string, it's treated as a straight React JSX template to be rendered.

#### `function`

If it's a function, the function will be called with the web server's `request` object, and it should return a promise that resolves an object:

```js
function myContent(req) {
  return Promise.resolve({
    status: 200,
    html: "<h1>Hello React!</h1>",
    prefetch: ""
  });
}
```

In an Electrode app, the module `electrode-redux-router-engine` and its `render` method is used to invoke the React component that's been specified for the route and the `renderToString` output is returned as the `html`.

#### `object`

If it's an object, it can specify a `module` field which is the name of a module that will be `require`ed.  The module should export either a string or a function as specified above.

### Disabling SSR

If you don't want any Server Side Rendering at all, you can set the option `serverSideRendering` to `false`, and you can skip setting `content`.  This will make the server to **not** even load your React app.

For example:

```js
const config = {
  plugins: {
    "electrode-react-webapp": {
      options: {
        pageTitle: "My Awesome React WebApp",
        paths: {
          "/{args*}": {
            view: "index"
        },
        unbundledJS: {
          enterHead: [
            {src: "http://cdn.com/js/lib.js"}
          ]
        },
        serverSideRendering: false
      }
    }
  }
};
```

[npm-image]: https://badge.fury.io/js/electrode-react-webapp.svg

[npm-url]: https://npmjs.org/package/electrode-react-webapp

[daviddm-image]: https://david-dm.org/electrode-io/electrode/status.svg?path=packages/electrode-react-webapp

[daviddm-url]: https://david-dm.org/electrode-io/electrode?path=packages/electrode-react-webapp

[daviddm-dev-image]: https://david-dm.org/electrode-io/electrode/dev-status.svg?path=packages/electrode-react-webapp

[daviddm-dev-url]: https://david-dm.org/electrode-io/electrode?path=packages/electrode-react-webapp?type-dev

[npm-downloads-image]: https://img.shields.io/npm/dm/electrode-react-webapp.svg

[npm-downloads-url]: https://www.npmjs.com/package/electrode-react-webapp
