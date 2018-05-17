# html-webpack-include-sibling-chunks-plugin

This plugin is useful when bundling a Multiple-Page Application with webpack 4.
It let `html-webpack-plugin` to include initial split chunks and runtime chunk that related to the entry js file of a html page,
which are generated by `optimization.splitChunks` and `optimization.runtimeChunk` (https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693).

Suppose your MPA has the following structure (check the [example](example) folder):

```
├── dist
├── package.json
├── node_modules
├── src
│   ├── components
│   ├── shared
|   ├── favicon.png
│   └── pages
|       ├── foo                      // http://localhost:8080/foo.html
|       |    ├── index.html
|       |    ├── index.js
|       |    ├── style.css
|       |    └── pic.png
|       └── bar                      // http://localhost:8080/bar.html
|           ├── index.html
|           ├── index.js
|           ├── style.css
|           └── baz                  // http://localhost:8080/bar/baz.html
|               ├── index.html
|               ├── index.js
|               └── style.css
└── webpack.config.js
```

You can bundle the site using this webpack config: (check the full file: [example/webpack.config.js](example/webpack.config.js))

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
const HtmlWebpackIncludeSiblingChunksPlugin = require('html-webpack-include-sibling-chunks-plugin')
const glob = require('glob')

const dev = Boolean(process.env.WEBPACK_SERVE)

const entries = glob.sync('./src/**/index.js')
const entry = {}
const htmlPlugins = []
for (const path of entries) {
  const template = path.replace('index.js', 'index.html')
  const chunkName = path.slice('./src/pages/'.length, -'/index.js'.length)
  // in order to hot reload in dev, we put template file into the entry
  entry[chunkName] = dev ? [path, template] : path
  htmlPlugins.push(new HtmlWebpackPlugin({
    template,
    filename: chunkName + '.html',
    chunksSortMode: 'none',
    chunks: [chunkName]
  }))
}

module.exports = {
  entry,

  optimization: {
    runtimeChunk: true,
    splitChunks: {
      chunks: 'all'
    }
  },

  plugins: [
    // ...
    // must be placed before html-webpack-plugin
    new HtmlWebpackIncludeSiblingChunksPlugin(),
    ...htmlPlugins
  ]

  // ...
}
```

`entry` and `htmlPlugins` will be generated by parsing the folder structure. for example:

entry:
```js
{
  'bar/baz': './src/pages/bar/baz/index.js',
  bar: './src/pages/bar/index.js',
  foo: './src/pages/foo/index.js'
}
```

htmlPlugins:
```js
[
  new HtmlWebpackPlugin({
    template: './src/pages/bar/baz/index.html',
    filename: 'bar/baz.html',
    chunksSortMode: 'none',
    chunks: ['bar/baz']
  },

  new HtmlWebpackPlugin({
    template: './src/pages/bar/index.html',
    filename: 'bar.html',
    chunksSortMode: 'none',
    chunks: ['bar']
  },

  new HtmlWebpackPlugin({
    template: './src/pages/foo/index.html',
    filename: 'foo.html',
    chunksSortMode: 'none',
    chunks: ['foo']
  }
]
```

This plugin will insert split chunks and runtime chunk that splitted from `chunks` of `HtmlWebpackPlugin` into the html file.

## Running the example project

```sh
cd example
npm install
npm run dev
```

Open http://localhost:8080/foo.html to check the result.

Building the production bundle:
```sh
npm run build
```
