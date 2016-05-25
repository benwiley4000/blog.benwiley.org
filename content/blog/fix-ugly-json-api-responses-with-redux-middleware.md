+++
contenttype = "article"
date = "2016-05-22T17:18:09-04:00"
draft = false
image = ""
slug = "fix-ugly-json-api-responses-redux-middleware"
title = "Fix Ugly JSON API Responses with Redux Middleware"

+++

***NOTE:*** *This post uses an example Redux application, but won't go deep into explaining how Redux actually works. Check out the (top-notch) [Redux documentation](http://redux.js.org/) to get started. However you won't be expected to know [React](https://facebook.github.io/react/), which is often used with Redux applications. For rendering in this example, we just use plain JavaScript.*

As a JavaScript app developer, you'll inevitably be handling JSON object responses from server APIs --- either your own, or someone else's. When you get lucky, the keys on those responses are already be `camelCased`. Other times, those keys may be formatted according to another convention, like `snake_case` or `PascalCase`.

At first this may seem like a small inconvenience. But it can become confusing to juggle variables names with the wrong casing style, especially if you've already ascribed a different meaning to variables with that style (for example, `PascalCase` variables normally refer exclusively to class names in a JavaScript app).

Wouldn't it be nice if you could tell your app to automatically `camelCase` all the keys in your download JSON responses? Good news --- I'm about to show you how.

This post is a two-parter. In part one, we're going to run through the construction of a very basic Redux app that requests a `snake_cased` JSON response, fetches it, and displays it on the screen. In part two, we'll enhance our app to automatically convert our response to `camelCase`!

## Part I: Building the app

***Note:*** *If you ever get confused about where something should go, you can check out the full example repository [here](https://github.com/benwiley4000/redux-json-request-formatting-tutorial).*

### Setting up the build environment

Before we write any code for our app, we'll want to make sure we have all our dependencies set up. First we'll want to make sure Node.js is installed on our machine. If you don't have it, you can [get it here](https://nodejs.org/).

Next we'll want to download the Node module dependencies for our project. Go ahead and create a new directory for the project. In that directory, create a file called `package.json`, and paste this into that file:

{{<highlight json>}}
{
  "devDependencies": {
    "babel-loader": "^6.2.4",
    "babel-preset-es2015": "^6.6.0",
    "babel-preset-stage-0": "^6.5.0",
    "webpack": "^1.13.0"
  },
  "dependencies": {
    "camelize": "^1.0.0",
    "redux": "^3.5.2",
    "redux-action-transform-middleware": "^0.4.1"
  }
}
{{</highlight>}}

Not familiar with all of these dependencies? [Redux](http://redux.js.org/), of course, is the framework we're using to organize the data in our app. The other two main `dependencies`, [camelize](https://www.npmjs.com/package/camelize) and [redux-action-transform-middleware](https://www.npmjs.com/package/redux-action-transform-middleware), are what we'll use later on to convert our JSON keys to `camelCase`.

All the [Babel](http://babeljs.io/)-related packages under `devDependencies` let us use the latest JavaScript syntax before modern web browsers support it. Babel transpiles our code to something browsers can understand. And [webpack](https://webpack.github.io/) takes all our downloaded dependencies and includes them in one big JavaScript file with our app, before we send it to the browser.

To download all of those modules, all you need to do is save your `package.json` file, then open your project's root directory in a terminal and run:

{{<highlight bash>}}
npm install
{{</highlight>}}

Next, we'll want to include the configuration that webpack will use to bundle our app and its dependencies together. Create another file called `webpack.config.js` in your project directory:

{{<highlight javascript>}}
var webpack = require('webpack');

var webpackConfig = {
  entry: './src/index.js',
  resolve: {
    extensions: ['', '.js']
  },
  output: {
    path: __dirname + '/dist',
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel',
        query: {
          presets: ['es2015', 'stage-0']
        }
      }
    ]
  }
};

module.exports = webpackConfig;
{{</highlight>}}

And save it.

Finally, we'll want to make sure our build process actually works! Create a new directory inside of your project directory called `src/`, and create `src/index.js`. Open it in your favorite text editor and type this:

{{<highlight jsx>}}
const someFunction = () => {
  return 'hooray!';
};
{{</highlight>}}

And hit save. Now, run the command `webpack` from your root project directory. There should now be a file called `dist/bundle.js`.

{{<highlight javascript>}}
'use strict';

var someFunction = function someFunction() {
  return 'hooray!';
};
console.log(someFunction());
{{</highlight>}}

### A tentative solution

No one wants to fill their code with statements like `var textContent = response.text_content`. Instead, there are existing modules out there that you can install with [`npm`](npmjs.com), that will transform your whole JSON response for you. A popular, and dead-simple one is [`camelize`](https://www.npmjs.com/package/camelize). Watch:

{{<highlight jsx>}}
import camelize from 'camelize';

const responseData = {
  data_root: {
    some_prop: {
      prop_a: 1,
      prop_b: 2
    },
    some_other_prop: [
      {
        item_a: 'x'
      },
      {
        item_b: 'y'
      }
    ]
  }
};

console.log(camelize(responseData));
/* prints:
 * { dataRoot: 
 *    { someProp: { propA: 1, propB: 2 },
 *      someOtherProp: [ [Object], [Object] ] } }
 */
{{</highlight>}}

If the keys in your original response are `PascalCase`, `camelize` won't be much help, but you can use a similar module called [`penrillian-camelize`](https://www.npmjs.com/package/penrillian-camelize) that adds support for converting from `PascalCase` to `camelCase`. If there's some other transform you need to apply to your keys, you can use [`recursive-json-key-transform`](https://www.npmjs.com/package/recursive-json-key-transform) (disclaimer: I published this module), which lets you specify your own function to apply to all the keys in your JSON response. You also may want to check out the module [`i`](https://www.npmjs.com/package/i), which contains a collection of common string transforms that can be used in conjunction with `recursive-json-key-transform`.



### Integrating this into an app data flow

Here's an app that fetches a JSON response (actually just a pre-built JSON object that pretends to be a JSON response), and prints it to the screen. You can press 'Fetch data' to fetch, and 'Clear data' to empty the data object. As you can see, the keys in the response are `snake_cased`.

<iframe src="http://benwiley4000.github.io/redux-json-request-formatting-tutorial/without_camelize.html" style="width:100%"></iframe>

First, let's dive into the pieces of this app.
