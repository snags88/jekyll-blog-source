---
layout: post
title: "Gulp: Fingerprinting Assets"
permalink: "gulp-fingerprinting-assets"
date: 2017-03-28 07:30:00 -0500
categories: web-development
---

The modern web browser does a lot of things behind the scenes to improve
the user experience, like caching asset files on your local disk. In
this way if we need to load them again, it just reuses those files
instead of making a request to the server.

> _As developers, we should know about caching because a.) it's free b.) it
makes our sites fast._

The way that caching client side assets works is pretty simple:

- An asset like `myJSFile.js` is referenced somewhere on your HTML doc
- The browser makes a request to get that asset from your server.
- If the same file name already exists on your local cache, it will
reuse that file and avoid making a request to the server.

There are some nuances with the response headers when the asset is
served that could prevent caching or set a cache duration. But for now,
we'll ignore those and assume all our assets are getting cached.

The common technique for "busting" the cache and forcing the client to
retrieve an updated asset is to "fingerprint" our files. As elegantly
explained by the [Rails Asset Pipeline doc](http://guides.rubyonrails.org/asset_pipeline.html#what-is-fingerprinting-and-why-should-i-care-questionmark): "Fingerprinting is a technique that makes the name of a file dependent on the contents of the file. When the file contents change, the filename is also changed. For content that is static or infrequently changed, this provides an easy way to tell whether two versions of a file are identical, even across different servers or deployment dates."

With those things in mind, let's dive into the meat of this post.

## Background
The problem that we encountered with Learn.co is that some of our
SVGs were not appearing when we deployed new SVGs until you cleared your
cache (or do a "hard" refresh).

After digging into it a bit, the problem is that the client side
templates were always referencing `/assets/sprite.svg` so when we added
new SVGs to our sprite file, the browser was using the old cached
version of the file.

This wasn't a problem with server-side rendered templates because Rails
auto-magically takes care of fingerprinting the asset files for us.
However, we have a pretty custom build process using Gulp for our client-side code
and it gets compiled before running through the asset pipeline.

Knowing the above, I came up with a solution to fingerprint files in
our Gulp build process.

## Implementation
The way I wanted to approach this was to fingerprint our assets, then
swap out references to it like good ol' Indiana Jones. Here are the
tools I used:

- [`gulp-rev`](https://github.com/sindresorhus/gulp-rev) will take in files and output fingerprinted versions + a manifest JSON file that maps the old filename
to the new filename.
- [`rev-del`](https://github.com/callumacrae/rev-del) will take in old
and new manifest JSON files and delete fingerprinted files that are no
longer being used.
- [`gulp-fingerprint`](https://github.com/vincentmac/gulp-fingerprint) will take a manifest JSON file and replace any
references to the old filenames with the new filenames.

I definitely ran into a few tricky parts so I'll detail each step here.

### Fingerprint our files
In our `gulpfile.js` we're going to create a task to fingerprint our
files. For simplicity sake, I'll hardcode the configurations.

{% highlight javascript %}
// gulpfile.js
var gulp = require('gulp')
  , gutil = require('gulp-util')
  , rev = require('gulp-rev')
  , revDel = require('rev-del')
  , through = require('through2')
  , path = require('path')
  ;

var fingerprintConfig = {
  src: './app/assets/images/sprite.svg', // to fingerprint
  dest: './app/assets/bin'               // output folder
}

// Define gulp task
gulp.task('fingerprint_assets', function() {
  return fingerprintAssets.run(fingerprintConfig);
})

// Fingerprinting process
var fingerprintAssets = {
  run: function run(config) {
    return gulp.src(config.src, {base: 'app/assets'}) //read files
      .pipe(rev())                                    //fingerprint files
      .pipe(gulp.dest(config.dest))                   //output to dest
      .pipe(rev.manifest())                           //create fingerprint manifest file
      .pipe(revDel({                                  //Remove old fingerprinted files
        oldManifest: './rev-manifest.json',
        dest: config.dest
      }))
      .pipe(gulp.dest('.'))                           //output manifest to root
      ;
  }
}
{% endhighlight %}

Not too bad right? Important points are:

- To get the file names correctly in accordance with Rails asset
pipeline. Your files will appear at `/assets/filename.ext`.
- You must the return the stream in our Gulp task, which forces any
dependencies on this task to be sychronous. By doing so, you force your
Javascript build to wait for assets to be fingerprinted.
- Lastly, you'll want to make sure the keys in the `rev-manifest.json`
files are correct in accordance with the Rails asset pipeline files.
Initially it will look like `{ "images/sprite.svg":
"images/sprite-9s3LDioc.svg" }`, but you want to replace your key to be
`assets/sprite.svg`. To do this I had to add a step to change the
manifest stream:

{% highlight javascript %}
// gulpfile.js
    // ...
    return gulp.src(config.src, {base: 'app/assets'})
      .pipe(rev())
      .pipe(gulp.dest(config.dest))
      .pipe(rev.manifest())
      .pipe(through2.obj(function(chunk, enc, cb) {   //update manifest file to correct keys for Rails pipeline
        var originalMap = JSON.parse(chunk.contents.toString(enc));

        var newMap = Object.keys(originalMap).reduce(function(mem, k) {
          var newFilePath = path.join('assets', path.basename(k))
          mem[newFilePath] = originalMap[k]
          return mem;
        }, {})

        chunk.contents = new Buffer(JSON.stringify(newMap, null, '  '))
        cb(null, chunk)
      }))
      .pipe(revDel({
        oldManifest: './rev-manifest.json',
        dest: config.dest
      }))
      .pipe(gulp.dest('.'))
      ;
    // ...
{% endhighlight %}

With the above task, we should get our fingerprinted files in
`./app/assets/bin/` and a manifest file at the root of our project
`./rev-manifest.json`. Also, anytime we change the file, the old
fingerprinted files are removed for us.

### Update references to the new file
The next step is to use the `./rev-manifest.json` file with
`gulp-fingerprint` to swap out the file references:

{% highlight javascript %}
// gulpfile.js
// omitting the dependencies for brevity

var jsBuildConfig = {
  src: './app/assets/javascripts/index.js',
  dest: './app/assets/javascripts/bin'
}

gulp.task('build', ['fingerprint_assets'], function () {
  jsBuild.bundle(jsBuildConfig)
});

var jsBuild = {
  bundle: function(config) {
    var bundler = browserify({entries: config.src}); // Note we're using browserify here.

    // Do some browserify transforms here
    bundler.transform('jadeify');
    bundler.transform('babelify', {presets: ['es2015', 'react']});

    //Read manifest to swap file references. Configure fingerprint
    options
    var manifest = JSON.parse(fs.readFileSync('./rev-manifest.json'))
      , fingerprintOpts = this._getFingerprintOpts(manifest)
      ;

    return bundler
      .bundle()
      .pipe(source(path.basename(file)))
      .pipe(buffer())
      .pipe(fingerprint(manifest, fingerprintOpts))
      .pipe(gulp.dest(config.dest))
      ;
  },

  _getFingerprintOpts: function _getFingerprintOpts(manifest) {
    // setup regex to match all manifest keys
    var regexString = _.map(Object.keys(manifest), function (file) { return '(' + file.replace('/', '\\/').replace('.', '\\.') +')' }).join('|')
      , regex = new RegExp(regexString)
      ;

    return {
      regex: regex,
      prefix: 'assets/',
      strip: 'images/',
      verbose: false // set this to true to debug
    }
  },
}
{% endhighlight %}

There's a lot of code there, so here's a quick explanation:

1. Set up configurations for the build task.
2. Set up the `'build'` task with a dependency on the
`fingerprint_assets` task.
3. Setup Browserify and bundle.
4. In the stream, use `gulp-buffer` to convert the stream to a buffer
   and pipe it into `gulp-fingerprint` to replace all references to
   `assets/sprite.svg` with the fingerprinted file reference.
  - Note the options uses the manifest to create a regex to match all
  occurances in our file.

## Results
To confirm that everything went well, we can open up our file in
`./app/assets/javascripts/bin/index.js` and make sure all the file
references are correct.

Now when you update the `sprite.svg` file, the fingerprinted filename
should change and bust any cache on the clients.

## Next step
A nice thing to do for our build process is to omit the
fingerprinting & replacing file references in our development
environment. You can do this by looking at `process.env.NODE_ENV` in our
`gulpfile.js` to conditionally run the fingerpriting steps.

That's all I got for now. Till next time, happy coding!
