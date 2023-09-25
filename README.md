# react-dynamic-css-plugin

Webpack plugin to transform dynamic react class names to short prefixed and obfuscated classes at runtime

## Installation

```bash
npm install react-dynamic-css-plugin --save-dev
```

## Usage

webpack.config.js

```js
const path = require("path");

const ReactDynamicCssPlugin = require("react-dynamic-css-plugin");

module.exports = {
	target: "web",
	entry: {
		main: "./src/index.jsx"
	},
	output: {
		path: path.join(__dirname, "build"),
		filename: `js/[name].min.js`,
		chunkFilename: "js/[name].min.js"
	},
	module: {
		rules: [
			{
				test: /\.(js|jsx)$/,
				resolve: {
					extensions: [".js", ".jsx"]
				},
				use: {
					loader: "babel-loader",
					options: {
						presets: ["@babel/preset-env", "@babel/preset-react"]
					}
				}
			},
			{
				test: /\.(css)$/,
				use: [
					{
						loader: "css-loader"
					}
				]
			}
		]
	},
	plugins: [
		new ReactDynamicCssPlugin({
			enabled: true,
			entryName: "main",
			localIdentName: "myPrefix_[md4:hash:base64:5]",
			attributes: /^(id|class)$/,
			exclusionTags: /(path)/i,
			exclusionValues: /^(css|sc|icon)-/i
		})
	]
};
```

## Options

-   **enabled** | Boolean | Default: true | If false, the plugin will not run
-   **entryName** | String | Default: undefined | The name of the entry point to inject the plugin. If _undefined_ will default to the first entry
-   **localIdentName** | String | Default: "[md4:hash:base64:5]" | The format of the generated class names as [(algorithm):hash:(digest):(length)]
-   **attributes** | RegExp | Default: /^(id|class)$/ | Regex for HTML attribute names to transform their value
-   **exclusionTags** | RegExp | Default: /(path)/i | Regex for HTML tag names to exclude from transformations
-   **exclusionValues** | RegExp | Default: /^(css|sc|icon)-/i | Regex for HTML attribute values to exclude from transformations

Note: The **exclusionValues** default excludes class names with the following prefixes. This is because they are commonly generated by other libraries and should not be transformed.

-   "css-" which is the [emotion](https://emotion.sh/) default
-   "sc-" which is the [styled-components](https://styled-components.com) default
-   "icon-" which is the the standard for icon classes

## Mechanism

1. The plugin sets **localIdentName** and **getLocalIdent** in the _css-loader_ options
2. The plugin replaces **setAttribute** in _react-dom_ with a custom function that transforms the class names at runtime
3. The plugin injects the custom **setAttributeDynamic** function into the entry point of the bundle
