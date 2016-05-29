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

{{<highlight json>}}
{
  "camelCase": {
    "propertyA": 1,
    "propertyB": 2
  },
  "snake_case": {
    "property_a": 1,
    "property_b": 2
  },
  "PascalCase": {
    "PropertyA": 1,
    "PropertyB": 2
  }
}
{{</highlight>}}

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
$ npm install
{{</highlight>}}

Next, we'll want to include the configuration that webpack will use to bundle our app and its dependencies together. Create another file called `webpack.config.js` in your project directory:

{{<highlight javascript>}}
/* webpack.config.js */
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
/* src/index.js */
const someFunction = () => {
  return 'hello, world!';
};
console.log(someFunction());
{{</highlight>}}

Hit save. From your project's root directory, run the command:
{{<highlight bash>}}
$ webpack
{{</highlight>}}

There should now be a file called `dist/bundle.js`, which should have something that looks like this near the end:

{{<highlight javascript>}}
/* dist/bundle.js */
'use strict';

var someFunction = function someFunction() {
  return 'hello, world!';
};
console.log(someFunction());
{{</highlight>}}

To be sure, go back to your terminal and run `node dist/bundle.js`. If everything is working, you shouldn't get any errors, and you'll see:

{{<highlight bash>}}
$ node dist/bundle.js
hello, world!
{{</highlight>}}

Great! All the dependencies are installed and working. If something failed, double check your directory tree to make sure it looks like this.

{{<highlight bash>}}
|- dist/
    |- bundle.js
|- node_modules/
    |- ... module directories ...
|- src/
    |- index.js
|- package.json
|- webpack.config.js
{{</highlight>}}

### Creating our HTML

In our project's root directory, let's create a new file called `index.html`. There's nothing fancy here --- just a container for our app and a couple of buttons.

{{<highlight html>}}

<!DOCTYPE html>
<!-- index.html -->
<html>
<head>
  <meta charset="utf-8" />
  <title>Demo: Formatting JSON Responses via Redux Middleware</title>
</head>
<body>
  <button id="fetch">Fetch data</button>
  <button id="clear">Clear data</button>
  <div id="data"></div>
  <script src="dist/bundle.js"></script>
</body>
</html>
{{</highlight>}}

### Putting together our Redux app

Go back to `src/index.js`, and clear everything out. Let's start by initializing our Redux store with a reducer and an initial state.

{{<highlight jsx>}}
/* src/index.js */

/* import dependencies */
import { createStore } from 'redux';

/* initial state */
const initialState = {
  data: {},
  fetching: false
};

/* reducer */
const reducer = (state, action) => {
  state = state || {};
  switch(action.type) {
    case 'DATA_REQUEST':
      return {
        ...state,
        fetching: true
      };
    case 'DATA_RESPONSE':
      return {
        ...state,
        data: action.res.data,
        fetching: false
      };
    case 'CLEAR_DATA':
      return {
        ...state,
        data: {}
      };
    default:
      return state;
  }
};

/* create store */
const store = createStore(reducer, initialState);
{{</highlight>}}

Let's also make sure our view reflects our store's current state.

{{<highlight jsx>}}
/* src/index.js */

/* import dependencies ... */

/* initial state ... */

/* reducer ... */

/* create store ... */

/* log current state to view */
const dataElement = document.getElementById('data');
dataElement.innerHTML = JSON.stringify(store.getState().data);

/* subscribe view to state updates */
store.subscribe(() => {
  const state = store.getState();
  if (state.fetching) {
    dataElement.innerHTML = '<i>Fetching...</i>';
  } else {
    dataElement.innerHTML = JSON.stringify(state.data);
  }
});
{{</highlight>}}

If you run `webpack` and open `index.html` in a browser, you should see our initial state (`{}`) in the window. But pressing the buttons doesn't do anything! We'll fix that by dispatching an action to the store when we click each button:

{{<highlight jsx>}}
/* src/index.js */

/* import dependencies ... */

/* initial state ... */

/* reducer ... */

/* create store ... */

/* log current state to view ... */

/* subscribe view to state updates ... */

/* dispatch DATA_REQUEST on 'Fetch data' */
document.getElementById('fetch').addEventListener('click', () => {
  store.dispatch({
    type: 'DATA_REQUEST'
  });
});

/* dispatch CLEAR_DATA on 'Clear data' */
document.getElementById('clear').addEventListener('click', () => {
  store.dispatch({
    type: 'CLEAR_DATA'
  });
});
{{</highlight>}}

Now clicking "Fetch" will trigger a message onscreen, but then the app will stall. That's because we're not actually fetching any data. A real data fetch could make this app more complicated than it needs to be for a demo, so we'll fake it.

For the request we'll use an advanced Redux feature called [middleware](http://redux.js.org/docs/advanced/Middleware.html). We apply the middleware to our store, and it will scan each action we dispatch to see if we're making a data request. If we are, it will continue dispatching our action that tells the store we're fetching, *and* after a delay, another action, carrying some made-up data.

{{<highlight jsx>}}
/* src/index.js */

/* import dependencies */
import { createStore, applyMiddleware } from 'redux';

/* initial state ... */

/* reducer ... */

/* fake response data */
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

/* response middleware */
const responseMiddleware = store => next => action => {
  const { type, ...rest } = action;

  if (type !== 'DATA_REQUEST') return next(action);

  next(action);
  setTimeout(() => {
    next({
      ...rest,
      type: 'DATA_RESPONSE',
      res: {
        data: responseData
      }
    });
  }, 700);
};

/* create store */
const store = createStore(
  reducer,
  initialState,
  applyMiddleware(responseMiddleware)
);

/* log current state to view ... */

/* subscribe view to state updates ... */

/* dispatch DATA_REQUEST on 'Fetch data' ... */

/* dispatch CLEAR_DATA on 'Clear data' ... */
{{</highlight>}}

### A note on middleware

If you've never used Redux middleware before, it's worth reading [the documentation](http://redux.js.org/docs/advanced/Middleware.html) to understand how it works. In short, it's something that can be dropped in between the dispatching of an action and the store's handling of that action to give your app enhanced functionality. If a piece of Redux middleware handles an action that meets its criteria, it typically dispatches a new action in place or in addition.

The middleware above would be pretty useless in a practical situation, but I based it off of [real middleware](https://github.com/choonkending/react-webpack-node/blob/a393fb34c17e73504560f8aadcaf7d121a9a1a40/app/middlewares/promiseMiddleware.js) you can use to await the asychronous result of a JavaScript Promise in your Redux app. If you're wondering how that hooks into the rest of an app, the [entire repo](https://github.com/choonkending/react-webpack-node) is full of great examples.

### Trying out our app

If you run `webpack` again, and open `index.html`, everything works. Clicking "Fetch data" updates our state (after a short delay) with our fake data, and "Clear data" resets our app.

<iframe src="http://benwiley4000.github.io/redux-json-request-formatting-tutorial/without_camelize.html" style="width:100%"></iframe>

**One problem:** *the keys are `snake_cased`!*

## Part II: camelCasing our data response

No one wants to fill their code with statements like `const textContent = response.text_content` as you parse through an entire object. Instead, there are existing modules out there that will transform your JSON response for you. A popular, and dead-simple one is camelize, which we already have installed. Watch:

{{<highlight jsx>}}
/* example */
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

### A note on other JSON transformation options

If the keys in your original response are `PascalCase`, camelize won't be much help, but you can use a similar module called [penrillian-camelize](https://www.npmjs.com/package/penrillian-camelize) that adds support for converting from `PascalCase` to `camelCase`. If there's some other transform you need to apply to your keys, you can use [recursive-json-key-transform](https://www.npmjs.com/package/recursive-json-key-transform) (disclaimer: I published this module), which lets you specify your own function to apply to all the keys in your JSON response. You also may want to check out the module [i](https://www.npmjs.com/package/i), which contains a collection of common string transforms that can be used in conjunction with `recursive-json-key-transform`.

### Integrating this into an app data flow

In our app, there's not necessarily an obvious place to use `camelize` if want to use it to transform our data responses. We know the data in our state needs to be `camelCased`, so one option would be to use `camelize` inside of our reducer as we update our state.

{{<highlight jsx>}}
/* example */
import camelize from 'camelize';

const reducer = (state, action) => {
  state = state || {};
  switch(action.type) {
    case 'DATA_RESPONSE':
      return {
        ...state,
        data: camelize(action.res.data),
        fetching: false
      };
    /* other cases ... */
  }
};
{{</highlight>}}

The problem with this becomes more evident as our reducer expands --- we don't want to have to remember to use `camelize` every time we write a reducer case that involves a data response, especially if we might end up using an API in the future that doesn't need to be transformed upon arrival.

Another option would be to use `camelize` inside our `responseMiddleware`, so we transform our JSON as soon as our app receives it.

{{<highlight jsx>}}
/* example */
import camelize from 'camelize';

const responseMiddleware = store => next => action => {
  const { type, ...rest } = action;

  if (type !== 'DATA_REQUEST') return next(action);

  next(action);
  setTimeout(() => {
    next({
      ...rest,
      type: 'DATA_RESPONSE',
      req: {
        data: camelize(responseData)
      }
    });
  }, 700);
};
{{</highlight>}}

This is actually a much better solution. Since our response middleware could be used to generalize for several action types, we only need to worry about using `camelize` in one place, and our reducer doesn't need to be coupled to our key transformation logic (which doesn't really have anything to do with the app state on its own).

But coupling `camelize` with our response middleware can also be problematic. Say we have multiple data sources, or we don't want to apply `camelize` to every single data response? It's better if we create a new piece of middleware dedicated to applying the transformation to our response. Redux allows us to chain together as many pieces of middleware as we want before the store handles our actions, so that shouldn't be a problem.

I've written a module called redux-action-transform-middleware. It's Redux middlware that can apply an arbitrary transformation to a given property on an action before the reducer receives it. In our case our data is nested at `res.data`, which is still a valid parameter.

In our app we can use redux-action-transform-middleware like so:

{{<highlight jsx>}}
/* src/index.js */

/* import dependencies */
import { createStore, applyMiddleware } from 'redux';
import actionTransformMiddleware from 'redux-action-transform-middleware';
import camelize from 'camelize';

/* initial state ... */

/* reducer ... */

/* fake response data ... */

/* response middleware ... */

/* camelize middleware */
const camelizeMiddleware = actionTransformMiddleware(
  'res.data',
  camelize
);

/* create store */
const store = createStore(
  reducer,
  initialState,
  applyMiddleware(responseMiddleware, camelizeMiddleware)
);

/* log current state to view ... */

/* subscribe view to state updates ... */

/* dispatch DATA_REQUEST on 'Fetch data' ... */

/* dispatch CLEAR_DATA on 'Clear data' ... */
{{</highlight>}}

If you're interested in understanding how `actionTransformMiddleware` works, you can see the full source (it's not very long) [right here](https://github.com/benwiley4000/redux-action-transform-middleware/tree/master/src/index.js).

### A working app.. without `snake_case`!

<iframe src="http://benwiley4000.github.io/redux-json-request-formatting-tutorial/" style="width:100%"></iframe>

And that's it. If we run `webpack`, open our app again in a browser, and hit "Fetch data," we'll see a `camelCased` response.

Questions? Comments? Feel free to leave them below!

### Links

[1] [redux-json-request-formatting-tutorial on GitHub](https://github.com/benwiley4000/redux-json-request-formatting-tutorial)

[2] [react-webpack-node on GitHub](https://github.com/choonkending/react-webpack-node) (full-stack app example using Redux, React, webpack, Node.js, latest ES2015 syntax)

[3] [Redux middleware documentation](http://redux.js.org/docs/advanced/Middleware.html)

[4] [redux-action-transform-middleware on npm](https://www.npmjs.com/package/redux-action-transform-middleware)

[5] [camelize on npm](https://www.npmjs.com/package/camelize)

[6] [penrillian-camelize on npm](https://www.npmjs.com/package/penrillian-camelize)

[7] [recursive-json-key-transform on npm](https://www.npmjs.com/package/recursive-json-key-transform)

[8] [i on npm](https://www.npmjs.com/package/i)
