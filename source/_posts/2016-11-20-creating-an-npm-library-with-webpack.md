---
layout: post
title: "Creating an NPM library with Webpack"
permalink: "creating-an-npm-library-with-webpack"
date: 2016-11-20 07:30:00 -0500
categories: web-development
---

I'm technically on vacation, but hey, it's all good. It's more relaxing
to learn about programming related stuff instead of going on massive
tours to Stonehenge. Anywho, I had a need to learn how to publish an NPM
library so I figured I'd jot down the steps I took to get a library
published. The library also uses Webpack to build the final source code.

It actually turned out way easier than I thought it was going to be. But
to make this post a little bit more concise, I'll assume that you
already have Node.js installed. If not, [here is NPM's guide](https://docs.npmjs.com/getting-started/installing-node)
 on how to do exactly that.

### Configuring NPM
The following commands will configure NPM on your local machine so that
your information will be added to the `package.json` file and published
from the correct NPM account:

{% highlight shell %}
$ npm set init.author.name "Optimus Prime"
$ npm set init.author.email "optimus.prime@autobots.com"
$ npm set init.author.url "http://optimus-prime.com"
$ npm adduser
{% endhighlight %}

The last command will either prompt you to create a new user in the npm
registry or login to an existing one.

### Initialize your project folder
First, create a git project directory with the basic npm configurations:

{% highlight shell %}
$ mkdir my-library && cd my-library
$ git init
$ npm init
{% endhighlight %}

The last command will start a mini-questionnaire to help fill out some
details on your `package.json` file. This file will contain the
configuration for npm to read when we are publishing the library.

This is what my first `package.json` file looks like. Don't worry, we'll
continue to develop this file:

{% highlight ruby %}
{
  "name": "my-library",
  "version": "1.0.0",
  "description": "Example library for my blog post",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "example"
  ],
  "author": "Seiji Naganuma <snaganumas@gmail.com> (https://snags88.github.io/)",
  "license": "MIT"
}
{% endhighlight %}

### Setup your project folder structure & files
We're going to need to setup our project folders and files. Keep in mind
that we're going to be using Babel as our transpiler, Webpack as our
bundler, and Mocha/Chai as our test runner/assertion library.

Here is our project folder structure and a short description of what each folder
or file is used for:

{% highlight text %}
lib/               => Output folder of our transpiled code
src/               => Our source code will all live in this folder
  index.js         => The main entry point for our bundler
test/              => Our specs will live here
  index.spec.js    => Our spec files should end in `*.spec.js`
.babelrc           => Our Babel configuration file
.gitignore         => Our project gitignore file
LICENSE            => Our license file
README.md          => Our readme file
package.json       => This file should have been created when we ran `npm init`
webpack.config.js  => This file will contain our Webpack configuration
{% endhighlight %}

### Installing our devDependencies
In the next few commands, we'll use npm to install all of our dev
dependencies and save them to our `package.json` file. Back in our
terminal, run the following command to install Babel + all its necessary
presents/plugins, Webpack, Mocha, and Chai.

{% highlight shell %}
$ npm install --save-dev babel babel-core babel-loader babel-plugin-add-module-exports babel-preset-es2015
$ npm install --save-dev webpack mocha chai
{% endhighlight %}

Note that I split it into two lines just because the command was getting
really long. This can be combined into 1 command if you want. I also
like to lock the minor version numbers of the libraries I installed so
I'll do that in my `package.json` file.
{% highlight ruby %}
{
  #...
  "devDependencies": {
    "babel": "6.5.x",
    "babel-core": "6.18.x",
    "babel-loader": "6.2.x",
    "babel-plugin-add-module-exports": "0.2.x",
    "babel-preset-es2015": "6.18.x",
    "chai": "3.5.x",
    "mocha": "3.1.x",
    "webpack": "1.13.x"
  }
  #...
}
{% endhighlight %}

### Configuring `.babelrc` and `.gitignore`
In `.babelrc` specify which presets and plugins you want to use. Here is
an example:

{% highlight ruby %}
{
  "presets": ["es2015"],
  "plugins": ["babel-plugin-add-module-exports"]
}
{% endhighlight %}

In `.gitignore` include all your favorite files to ignore. But make sure
you include the `node_modules` folder. Otherwise, all your dependencies
will be checked into your repo.

{% highlight ruby %}
# Dependency directories
node_modules
{% endhighlight %}

### Configuring Webpack
We will quickly update our `webpack.config.js` to do a few things:

- Know where to find our entry file (`src/index.js`)
- Where to output our transpiled file (`lib/my-library.js`)
- Know what loader to use to transpile our code. Babel in our case.
- Minify the file depending on the build environment.

Here is what my `webpack.config.js` looks like. I won't get into details
of how each thing works, but you can easily look it up on the Webpack
documentation.

{% highlight js %}
var webpack        = require('webpack')
  , path           = require('path')
  , UglifyJsPlugin = webpack.optimize.UglifyJsPlugin
  , env            = process.env.WEBPACK_ENV
  ;

var libraryName = 'my-library'
  , outputFile  = ''
  , plugins     = []
  ;

// configure output for proper build type
if (env === 'build') {
  plugins.push(new UglifyJsPlugin({ minimize: true }));
  outputFile = libraryName + '.min.js';
} else {
  outputFile = libraryName + '.js';
}

module.exports = {
  entry: path.join(__dirname, 'src','index.js'),
  output: {
    path: path.join(__dirname, 'lib'),
    filename: outputFile,
    library: libraryName,
    libraryTarget: 'umd',
    umdNamedDefine: true
  },
  module: {
    loaders: [
      {
        test: /(\.jsx|\.js)$/,
        loader: 'babel',
        exclude: /(node_modules)/
      }
    ]
  },
  resolve: {
    root: path.resolve('./src'),
    extensions: ['', '.js']
  }
}
{% endhighlight %}

### Updating our `package.json`
Since we've made changes to our Webpack configuration, let's make sure
that our `package.json` is up-to-date.

First, let's change the `main` entry to point to the correct file
`lib/my-library`. This is what will be ultimately used by anyone using
our library. See the documentation [here](https://docs.npmjs.com/files/package.json#main).

{% highlight ruby %}
{
  #...
  "main": "lib/my-library.js",
  #...
}
{% endhighlight %}
Next, we'll want to add some `script`s mostly to help bundle our code in
dev + production and to run our tests.

{% highlight ruby %}
{
  #...
  "scripts": {
    "build": "WEBPACK_ENV=build webpack",
    "dev": "WEBPACK_ENV=dev webpack --progress --colors --watch",
    "test": "mocha --compilers js:babel-core/register --colors -w ./test/*.spec.js"
  },
  #...
}
{% endhighlight %}

Now we can run `$ npm run dev` or `$ npm run build` to bundle our code.
We can also run `$ npm test` to run any specs in `test/*.spec.js`.

### Publishing to NPM
After you've written your library, some specs, and filled out the `README.md` file
with some useful information, you're ready to publish the library. One
useful thing to do before you publish is to make sure everything works
by installing your current node module globally and testing it in the
node repl:

{% highlight shell %}
$ npm install . -g
$ node
> var myLib = require('myLib');
> // use your library here to make sure it works.
{% endhighlight %}

Once you've done that, you can publish it with a really simple command:

{% highlight shell %}
$ npm publish
{% endhighlight %}

And that's it! Your library is published for the whole world to use.
Just install it using `$ npm install my-library`.

### Conclusion
Publishing your own libraries to npm is not a hard as I first thought it
would be. Since we come across similar problems in web development, it's
nice to share some of the solutions you've implemented. If you'd like to see the finished repo, you can find it [here](https://github.com/snags88/webpack-starter).

In the next post, I'll be talking about the implementation behind the first library I published to npm.
