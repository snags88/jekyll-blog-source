---
layout: post
title: "Basic React Project Configuration"
permalink: "basic-react-project-configuration"
date: 2016-05-08 13:04:00 -0500
categories: web-development
---
I made it through the [basic React-Rails tutorial](https://www.airpair.com/reactjs/posts/reactjs-a-guide-for-rails-developers) and read through some documentation on the [Facebook React page](https://facebook.github.io).
Naturally, feeling pretty good about myself, I charged ahead to make a
simple ToDo list using React.js. I mean super simple as in no data
persistence, no styling, no fancy animations.

A few hours later, I still haven't written a single line of code related
to React... Main reason is that the tutorial I ran through didn't really
talk about setting up a React project from scratch. There was a lot of
nice abstractions built with the `react-rails` gem and all the
behind-the-scenes Rails magic.

After some researching I guess there are a few ways of accomplishing
this. For this particular setup, we'll be using

1. `npm` as our package manager
2. `browserify` to modularize our code to be used across different files
3. `gulp` as our build/task-runner.

I did read that [Webpack](https://webpack.github.io/) is what most people are using now a days,
but I'll tackle the configuration for Webpack in a different post.

## Setting up our project folder

So let's assume that we already have our project folder and it's
connected to a git repo. Structure your project folder so it looks like
this:

{% highlight text %}
src/
  |-js/
    |-components/
    |-App.jsx
  |-index.html
{% endhighlight %}

This source folder will hold our code. Important note here is that our
`App` file is using the `.jsx` extension. This will make more sense when
we actually start writing code related to React, but for now just know
that it's a Javascript precompiler that allows you to write HTML-like
code into your js file... kinda like coffeescript.

## Node Package Manager

The first thing we're going to setup is NPM. You can do this by running
`npm init`. It will ask a bunch of questions about the project so that
all of the information can be stored in the `package.json` file in the
root of our project.

The `package.json` file is like a blueprint that describes our
project and its dependencies. This way, if we put this repo in another
environment, you can just run `npm install` to get all of the
dependencies. For more info about `package.json` check out the
[documentation here](https://docs.npmjs.com/files/package.json).

Another thing you might want to do here is to add a `.gitignore` file to
the root of your project and add `node_modules/*` to the file. All the
node packages we install will end up in that folder and we don't want to
commit those to git.

## Configuring Browserify

Ok, now that we have a `package.json` file, let's start installing some
npm modules. The first one we'll install is `browserify`. This bad boy
will let us modularize our code in different files and require them into
other files. You can read more about Browserify
[here](http://browserify.org/).

To install Browserify, run: `$ npm install browserify --save-dev` in your CLI.

This will install browserify for our project AND save it as a
`devDependency` in our `package.json` file. If you open up your
`package.json` file you'll see something like:

{% highlight json %}
"devDependencies": {
  "browserify": "^13.0.1"
}
{% endhighlight %}

The next one we want is `babelify`. This is a transform used by
browserify to "Babelify" our code. [Here](https://www.youtube.com/watch?v=Uk2bgp8OLT8)
is a good video on what a browserify transform does. And if you're
wondering what "Babel" is, it's a javascript compiler that takes files
like `*.jsx` and compiles it into proper javascript files. Another
reason for using Babel is that you can actually write ES2015 syntax and
it will compile down to ES5, which is supported by all modern web
browsers. For more about Babel, check out their [site](https://babeljs.io/)

Ok, to install Babelify, run: `$ npm install babelify --save-dev`.

We also want to install some presets so that babel can properly compile
our files. The two we need are for ES2015 and react.

Run: `$ npm install babel-preset-es2015 babel-preset-react --save-dev`.

Lastly, create a file called `.babelrc` in the root of your project and
stick the following json in there to make sure these presets are used:

{% highlight json %}
{
  'presets': ['es2015', 'react']
}
{% endhighlight %}

## Requiring React.js

There are a few ways to require the React.js library. We're just going
to do a quick and dirty script tag pointing to a CDN. You can copy and
paste this into your CLI and it will copy down some HTML I've already
written that contains the script tag for the React library.

{% highlight text %}
$ curl https://gist.githubusercontent.com/snags88/200fe956ec10a829c4757f11ecef4c84/raw/85a98137b3023561625df06947ed624bdae0a972/react-basic-index.html > src/index.html
{% endhighlight %}

Other options you have for grabbing the React library include installing
it as an NPM module.

## Using Babelify

Alright, now we can put all this stuff to use. Create a file called
`hello.jsx` inside the components folder and put some simple code
in the file:

{% highlight js %}
// src/js/components/hello.jsx

var Hello = React.createClass({
  render: function render () {
    return (
      <h1> Hello world</h1>
    );
  }
})

module.exports = Hello;
{% endhighlight %}

That `module.exports` will export out the `Hello` variable. When the
file is required in another, whatever was exported will be available. So
in our `App.jsx` file we can render this React component

{% highlight js %}
// src/js/App.jsx

var HelloComponent = require('./components/hello.jsx');

ReactDOM.render(<HelloComponent />, document.getElementById('js--content'));
{% endhighlight %}

Some of the syntax used in this file is definitely not legit javascript,
but when the code is built, it will compile properly.

## Using Gulp as our builder

In order to use Gulp, we'll have to install it as a dependency in our
project. We'll also use `gulp-browserify` to run browserify in our gulp
task.

`$ npm install gulp gulp-browserify gulp-rename --save-dev`

We will define our Gulp tasks in a file called `gulpfile.js`. The way
that tasks are defined is that we can read a file, then pipe it through
to do what you will such as renaming it, compiling it, or copying it to
a different folder. In my `gulpfile.js` I created a task to compile my
`.jsx` file and also copy it over to my `public/` and `build/` files.


{% highlight js %}
// gulpfile.js

//require our npm modules
var gulp = require('gulp')
  , browserify = require('gulp-browserify')
  , rename = require('gulp-rename')
  ;

//create js compile task
gulp.task('build-js', function() {
  return gulp.src('src/js/app.jsx')
    .pipe(browserify({
      transform: ['babelify'],
      extensions: ['.jsx']
    }))
    .pipe(rename('app.js'))
    .pipe(gulp.dest('build'))
    .pipe(gulp.dest('public/js'));
});

//create html compile task
gulp.task('build-html', function() {
  return gulp.src('src/index.html')
    .pipe(gulp.dest('public'));
});

//create helper tasks
gulp.task('build', ['build-js', 'build-html']);
gulp.task('default', ['build']);
{% endhighlight %}

Now, if we run `$ gulp`, it will kick off our default task and build our
js and html files and put them in the appropriate place.

Another thing we'll do here is to create a watch task so that any
changes in our js or html file will force the build task to run again.
Add the folowing to the `gulpfile.js` and update our default task.

{% highlight js %}
gulp.task('watch', function() {
  gulp.watch('src/js/**/*.jsx', ['build-js']);
  gulp.watch('src/js/*.jsx', ['build-js']);
  gulp.watch('src/index.html', ['build-html']);
});

...

gulp.task('default', ['build', 'watch']);
{% endhighlight %}

Now when you run `$ gulp`, that task will continue running and watch for
any changes in the files specified. To test this out, open the
`public/index.html` file. You should see `Hello World` on our page
that's rendered from our React Component. Now, if you go to your source
code and change the contents of `hello.jsx` to read `Hello World!!!` and
save, the code will be automatically be rebuilt. Refresh the
`index.html` page to confirm that the change in the jsx file is
reflected in the post-build html page.

## Wrapping up

Whew, that was a pretty long process. Hopefully that gave you a step by
step of the configuration needed to get set up to use React with
Browserify + Gulp. Now I'm
going to go play around and get more comfortable with React. Check out
the final version of this code in this [repo](https://github.com/snags88/react-basic-config).
