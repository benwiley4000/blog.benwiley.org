+++
contenttype = "article"
date = "2016-03-22T20:11:59-04:00"
draft = false
image = ""
slug = "using-react-create-audio-player-loops-through-playlists"
title = "Using React to Create an Audio Player that Loops Through Playlists"

+++

In a previous life I thought I would grow up to be a professional jazz musician. That dream sort of died, but I continue to write music, particularly for the [web browser-based video games](http://benwiley.org/#/games) I've worked on in recent months. When I was working on my personal site I figured it could be cool to host and play some of my own music for site visitors, so I built an audio player with JavaScript and CSS to do just that.

Pretty soon I realized others might want to use something like this in their own projects, so I spun the audio player out into a [**React**](https://facebook.github.io/react/) component module that you can now [install from npm](https://www.npmjs.com/package/react-responsive-audio-player). And while working on *that*, it occurred to me that the process of building that component could be a fun excuse for a blog post!

This tutorial assumes no prior knowledge of React, the build tools we'll be using (Node.js, npm, Babel, gulp, etc.), or even CSS (though some basic knowledge will help you understand what's going on). If you have a build setup or coding style you prefer, feel free to adjust accordingly. And if you have any questions about choices that are made, feel free to ask them below in the comments!

### Contents

* [Setting up the workspace]({{<relref "react-audio-player.md#setting-up-the-workspace">}})
* [Creating our component boilerplate]({{<relref "react-audio-player.md#creating-our-component-boilerplate">}})
* [Fleshing out our render markup]({{<relref "react-audio-player.md#fleshing-out-our-render-markup">}})

### Setting up the workspace

The first thing you'll need to do is make sure you have Node.js installed on your machine. [Go here to install it](https://nodejs.org/en/), if you haven't already.

Before we can begin writing our React component we'll need to install dependencies and set up our build process. Since we're going to code our React component using **ES2015** and **JSX** syntax, which isn't readable in today's web browsers, we'll need to **transpile** our code using a tool called [**Babel**](http://babeljs.io/). While ES2015 will be in most web browsers soon, JSX is a special HTML-like syntax created by the React team, for defining component structure. It will probably never be available in web browsers, but the [React preset for Babel](http://babeljs.io/docs/plugins/preset-react/) lets us write in JSX then convert it to normal-looking JavaScript before we render our work on the web.

To get started, create a `package.json` file in your project's directory, and populate it with an empty JSON object:

{{<highlight json>}}
{}
{{</highlight>}}

Let's install all our dependencies with [**npm**](https://www.npmjs.com/) (which comes bundled with Node).

First install the [**gulp**](http://gulpjs.com/) command line interface globally.

{{<highlight bash>}}
$ npm install -g gulp-cli
{{</highlight>}}

Next install the rest of our development dependencies locally.

{{<highlight bash>}}
$ npm install --save-dev babel-preset-es2015 babel-preset-react babelify browserify gulp gulp-autoprefixer gulp-concat gulp-sass vinyl-source-stream
{{</highlight>}}

We can also go ahead and install all the runtime dependencies we'll need to include in our final bundle.

{{<highlight bash>}}
$ npm install --save classnames react react-dom
{{</highlight>}}

Take a look back at our `package.json`. It's now filled with a list of our dependencies.

{{<highlight json>}}
{
  "dependencies": {
    "classnames": "^2.2.3",
    "react": "^0.14.7",
    "react-dom": "^0.14.7"
  },
  "devDependencies": {
    "babel-preset-es2015": "^6.6.0",
    "babel-preset-react": "^6.5.0",
    "babelify": "^7.2.0",
    "browserify": "^13.0.0",
    "gulp": "^3.9.1",
    "gulp-autoprefixer": "^3.1.0",
    "gulp-concat": "^2.6.0",
    "gulp-sass": "^2.2.0",
    "vinyl-source-stream": "^1.1.0"
  }
}
{{</highlight>}}

You should also notice our dependency files have been placed into a new directory called `node_modules/`.

We can now go ahead and write our `gulpfile.js`, which defines how gulp will turn our source files into browser-ready files (all these files and directories will exist soon). It's below, but if you don't know how gulp works and you're curious, [read this](https://css-tricks.com/gulp-for-beginners/).

{{<highlight javascript>}}
var gulp = require('gulp');
var browserify = require('browserify');
var concat = require('gulp-concat');
var sass = require('gulp-sass');
var source = require('vinyl-source-stream');
var autoprefixer = require('gulp-autoprefixer');

gulp.task('copyHTML', function () {
  return gulp.src('./src/index.html')
    .pipe(gulp.dest('./public/'));
});
 
gulp.task('browserify', function() {
  return browserify('./src/audioplayer.js')
    .transform('babelify', { presets: ['es2015', 'react'] })
    .bundle()
    .pipe(source('bundle.js'))
    .pipe(gulp.dest('./public/'));
});

gulp.task('sass', function () {
  return gulp.src('./src/*.scss')
    .pipe(concat('style.scss'))
    .pipe(sass().on('error', sass.logError))
    .pipe(autoprefixer({ browsers: ['> 2%'] }))
    .pipe(gulp.dest('./public/'));
});

gulp.task('default', ['copyHTML', 'browserify', 'sass']);
{{</highlight>}}

If you're paying attention, you noticed we're using [**Browserify**](http://browserify.org/), which lets us use Node modules in our JavaScript code, and [**Sass**](http://sass-lang.com/), which lets us use SCSS syntax before we transpile to normal CSS. We're also using Babel for transforming back to ES5 JavaScript, and [**Autoprefixer**](https://github.com/postcss/autoprefixer), which handles CSS vendor prefixes for us, so we can pretend they don't exist.

Let's update our `package.json` with a `build` script, which will run our gulp tasks and re-create our `public/` directory. We can execute that script with `npm run build`.

{{<highlight json>}}
{
  "dependencies": {
    ...
  },
  "devDependencies": {
    ...
  },
  "scripts": {
    "build": "rm -rf public/ && gulp"
  }
}
{{</highlight>}}

Finally, since I'm going to be storing my changes in a git repository, I'm going to create a `.gitignore` file to filter out the `node_modules/` directory, which I can re-create with `npm install`, and the `public/` directory, which I can create by building. Redundancy in repositories should be avoided when possible!

{{<highlight bash>}}
node_modules/
public/
{{</highlight>}}

If you've been following along, your directory structure should now look like this:

{{<highlight bash>}}
|- node_modules/
    |- ... module directories ...
|- .gitignore
|- gulpfile.js
|- package.json
{{</highlight>}}

### Creating our component boilerplate

To make sure everything's working, let's render a basic React component in a web browser.

First, create a new directory called `src/`, and place our `style.scss` in it. We won't be going over it, but we will need it in order for anything to show up. <a href="audioplayer.scss" download="style.scss">Download it here</a> (if you have any questions, let me know in the comments).

Next, create our `index.html` file, including a div with the `id` `audio_player_container`, into which we'll insert our React component.

{{<highlight html>}}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>React Audio Player</title>
    <style>
      html, body {
        /* margin: 0 ensures audio player will
         * take up full screen width.
         */
        margin: 0;
        background-color: lightgreen;
      }
    </style>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <div id="audio_player_container"></div>
    <script src="bundle.js"></script>
  </body>
</html>
{{</highlight>}}

Finally, we'll lay down our React component boilerplate in a file called `audioplayer.js`.

{{<highlight jsx>}}
const React = require('react');
const ReactDOM = require('react-dom');
const classNames = require('classnames');

class AudioPlayer extends React.Component {

  render () {
    return (
      <div className="audio_player"></div>
    );
  }

}

ReactDOM.render(
  <AudioPlayer/>,
  document.getElementById('audio_player_container')
);
{{</highlight>}}

Now that all that's in place, let's run `npm run build`. If you see error messages, you did something wrong. If not, great! The build was successful. Your directory structure should now look like this:

{{<highlight bash>}}
|- node_modules/
    |- ... module directories ...
|- public/
    |- bundle.js
    |- index.html
    |- style.css
|- src/
    |- audioplayer.js
    |- index.html
    |- style.scss
|- .gitignore
|- gulpfile.js
|- package.json
{{</highlight>}}

To make sure everything worked, go ahead and open `public/index.html` in a web browser. If you see a green background with a dark-colored bar at the bottom of the screen, you're good.

![This screen is good](img/2016-03-23boilerplate.jpg)

### Fleshing out our render markup