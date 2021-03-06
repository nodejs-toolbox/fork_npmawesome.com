---
layout: post
partner: nodejitsu
date: 2014-02-19T09:26:59-0800
tags: gulp.js
slug: 9-gulp-js-plugins-for-a-great-build-system
title: 9 gulp.js plugins for a great build system
description: 9 gulp.js plugins for a great build system
---
This article assumes you have experience with node.js, npm and you are pretty familiar with the concept of task runners and command-line interface. It will walk through general steps required to get up and running with [gulp.js].

Not unlike all the other build systems, it all starts by installing the task runner itself, e.g. `npm install gulp` and creating the main file in the root of your project called `gulpfile.js`. For those people like myself, who immediately want to know if [gulp.js] supports CoffeeScript - the answer is yes - just `require('./gulpfile.coffee')` from `gulpfile.js`.

Now that we have everything sorted out, lets get cracking and make a task runner to support our HTML5 node.js project.

<%- @readMore() %>

## [gulp-util]

Source: <%- @reference({ name: 'gulp-util', repo: 'gulpjs/gulp-util', license: 'MIT' }) %>

This is the official tool belt for [gulp.js]. The authors decided to split up helpers into a separate module which you basically end up requiring every time. This module brings in functionality for logging, coloring console output, and so on. Check out the [gulp-util] github page for the full list.

Since this is a utility module, there isn't a specific place to get started with it, so we will use it throughout instead.

## [gulp-clean]

Source: <%- @reference({ name: 'gulp-clean', repo: 'peter-vilja/gulp-clean', license: 'MIT' }) %>

The first thing any self-respecting build system should do is clean the space and remove everything that's in the way.

    var gulp = require('gulp');
    var clean = require('gulp-clean');

    gulp.task('clean', function () {
      return gulp.src('build', {read: false})
        .pipe(clean());
    });

You can now run `gulp clean` and the `build` folder in the root of your project will be obliterated.

## [gulp-concat]

Source: <%- @reference({ name: 'gulp-concat', repo: 'wearefractal/gulp-concat', license: 'MIT' }) %>

It's probably safe to assume that your HTML5 project has a few external dependencies like [jQuery](http://jquery.com), maybe [Modernizr](http://modernizr.com/) and such. Instead of having a separate `<script>` tag for each one of them, it would be nice to concat them together into one file. That's where [gulp-concat] comes in:

    var concat = require('gulp-concat');

    gulp.task('vendor', function() {
      return gulp.src('vendor/*.js')
        .pipe(concat('vendor.js'))
        .pipe(gulp.dest('build/vendor.js'))
    });

Now, running `gulp vendor` will take all `*.js` files from the local `vendor` folder and concat them into `build/vendor.js`.

## [gulp-uglify]

Source: <%- @reference({ name: 'gulp-uglify', repo: 'terinjokes/gulp-uglify', license: 'MIT' }) %>

The next thing to do is to minify our `vendor.js`. Lets add this to our `vendor` task.

    var uglify = require('gulp-uglify');

    gulp.task('vendor', function() {
      return gulp.src('vendor/*.js')
        .pipe(concat('vendor.js'))
        .pipe(uglify())
        .pipe(gulp.dest('build/vendor.js'))
    });

Notice, that instead of creating a separate task for minifying, we simply added another `pipe` call. This is essentially what [gulp.js] is all about - applying the right tools!

This is also where things might start going wrong, because [gulp-uglify] will throw an error if you have invalid JavaScript syntax. To deal with that lets add a basic error:

    var gutil = require('gulp-util');

    gulp.task('vendor', function() {
      return gulp.src('vendor/*.js')
        .pipe(concat('vendor.js'))
        .pipe(uglify())
        .pipe(gulp.dest('build/vendor.js'))
        .on('error', gutil.log)
    });

## [gulp-rename]

Source: <%- @reference({ name: 'gulp-rename', repo: 'hparra/gulp-rename', license: 'MIT' }) %>

Do you want to keep both, minified and source files around? That's not a problem! Lets extend our `vendor` task to write out `vendor.js` and `vendor.min.js` at the same time.

    var rename = require('gulp-rename');

    gulp.task('vendor', function() {
      return gulp.src('vendor/*.js')
        .pipe(concat('vendor.js'))
        .pipe(gulp.dest('build'))
        .pipe(uglify())
        .pipe(rename('vendor.min.js`))
        .pipe(gulp.dest('build'))
        .on('error', gutil.log)
    });

Notice that `gulp.dest` is used twice here. This is the cool thing about [gulp.js] - everything is just streams. Adding `gulp.dest` merely dumps whatever we currently have in the stream to disk, we can then mutate it further and save the new state again. Cool, eh?

## [gulp-filesize]

Source: <%- @reference({ name: 'gulp-filesize', repo: 'Metrime/gulp-filesize', license: 'MIT' }) %>

Don't know about you, but after minifying JavaScript I always want to know the file size. Guess what - I'm not the only one and there's a plugin for that.

    var filesize = require('gulp-filesize');

    gulp.task('vendor', function() {
      return gulp.src('vendor/*.js')
        .pipe(concat('vendor.js'))
        .pipe(gulp.dest('build'))
        .pipe(filesize())
        .pipe(uglify())
        .pipe(rename('vendor.min.js`))
        .pipe(gulp.dest('build'))
        .pipe(filesize())
        .on('error', gutil.log)
    });

Again, see how `filesize` is used twice? It will first print out the size of our source file and then the minified size.

## [gulp-less]

Source: <%- @reference({ name: 'gulp-less', repo: 'plus3network/gulp-less', license: 'MIT' }) %>

Lets assume you used the amazing [LESS] preprocessor for your generating your CSS files and now want to generate them. [gulp-less] to the rescue!

    var less = require('gulp-less');
    var path = require('path');

    gulp.task('css', function () {
      return gulp.src('less/**/*.less')
        .pipe(less({
          paths: [ path.join(__dirname, 'less', 'includes') ]
        }))
        .pipe(gulp.dest('build/css'))
        .on('error', gutil.log);
    });

Running `gulp css` will compile all [LESS] files from the `less` folder into `build/css`.

## [gulp-changed]

Source: <%- @reference({ name: 'gulp-changed', repo: 'sindresorhus/gulp-changed', license: 'MIT' }) %>

I would be forever unhappy if every time I ran `gulp css` all of my files would be regenerated regardless if the source [LESS] files have changed or not. Lets use [gulp-changed] to excluded not modified files.

    var changed = require('gulp-changed');

    gulp.task('css', function () {
      return gulp.src('less/**/*.less')
        .pipe(changed('build/css'))
        .pipe(less({
          paths: [ path.join(__dirname, 'less', 'includes') ]
        }))
        .pipe(gulp.dest('build/css'))
        .on('error', gutil.log);
    });

## [gulp-watch]

Source: <%- @reference({ name: 'gulp-watch', repo: 'floatdrop/gulp-watch', license: 'MIT' }) %>

It would be really cool if you didn't have to run `gulp css` every time you make a change, right? Lets set up a task that will monitor our files for changes and and compile them right away.

[gulp-watch] is a little bit different from the other plugins and we use it instead of `gulp.src` as the starting point.

    var watch = require('gulp-watch');

    gulp.task('css:watch', function () {
      watch({
        glob: 'less/**/*.less',
        emit: 'one',
        emitOnGlob: false
      }, function(files) {
        return files
          .pipe(less({
            paths: [ path.join(__dirname, 'less', 'includes') ]
          }))
          .pipe(gulp.dest('build/css'))
          .on('error', gutil.log);
      });
    });

`gulp css:watch` will being watching all of our LESS files and compile only the changed one.

## All together

Now lets put it all together into `gulpfile.js`

    var path = require('path');
    var gulp = require('gulp');
    var gutil = require('gulp-util');
    var clean = require('gulp-clean');
    var concat = require('gulp-concat');
    var uglify = require('gulp-uglify');
    var rename = require('gulp-rename');
    var filesize = require('gulp-filesize');
    var less = require('gulp-less');
    var changed = require('gulp-changed');
    var watch = require('gulp-watch');

    gulp.task('clean', function () {
      gulp.src('build', {read: false})
        .pipe(clean());
    });

    gulp.task('vendor', function() {
      return gulp.src('vendor/*.js')
        .pipe(concat('vendor.js'))
        .pipe(gulp.dest('build'))
        .pipe(filesize())
        .pipe(uglify())
        .pipe(rename('vendor.min.js`))
        .pipe(gulp.dest('build'))
        .pipe(filesize())
        .on('error', gutil.log)
    });

    gulp.task('css', function () {
      return gulp.src('less/**/*.less')
        .pipe(changed('build/css'))
        .pipe(less({
          paths: [ path.join(__dirname, 'less', 'includes') ]
        }))
        .pipe(gulp.dest('build/css'))
        .on('error', gutil.log);
    });

    gulp.task('css:watch', function () {
      watch({
        glob: 'less/**/*.less',
        emit: 'one',
        emitOnGlob: false
      }, function(files) {
        return files
          .pipe(less({
            paths: [ path.join(__dirname, 'less', 'includes') ]
          }))
          .pipe(gulp.dest('build/css'))
          .on('error', gutil.log);
      });
    });

## Summary

The thing that I personally really like about [gulp.js] is that it feels to me like a toolbox full of single purpose tools that I can use to assemble almost anything. It's amazing how in a span of a couple of months community has wrote [over 300 plugins](http://gulpjs.com/plugins/).

[LESS]: http://lesscss.org/
[gulp.js]: http://gulpjs.com
[gulp-util]: https://github.com/gulpjs/gulp-util
[gulp-clean]: https://github.com/peter-vilja/gulp-clean
[gulp-concat]: https://github.com/wearefractal/gulp-concat
[gulp-uglify]: https://github.com/terinjokes/gulp-uglify
[gulp-rename]: https://github.com/hparra/gulp-rename
[gulp-filesize]: https://github.com/Metrime/gulp-filesize
[gulp-less]: https://github.com/plus3network/gulp-less
[gulp-changed]: https://github.com/sindresorhus/gulp-changed
[gulp-watch]: https://github.com/floatdrop/gulp-watch