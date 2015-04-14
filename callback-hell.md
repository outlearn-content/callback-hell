<!--
name: callback-hell
version : 0.0.1
title : "Callback Hell"
description: "A guide to writing asynchronous javascript programs."
homepage : "http://callbackhell.com"
author : "Max Ogden"
license : "Unknown",
"contact" : {
  "url" : "http://maxogden.com/",
  "twitter": "@maxogden"
}
-->

<!-- @section -->

# What is "callback hell"?

Asynchronous javascript, or javascript that uses callbacks, is hard to get right intuitively. A lot of code ends up looking like this:

``` javascript
fs.readdir(source, function(err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function(filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function(err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = (values.width / values.height)
          widths.forEach(function(width, widthIndex) {
            height = Math.round(width / aspect)
            console.log('resizing ' + filename + 'to ' + height + 'x' + height)
            this.resize(width, height).write(destination + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
})
```

See all the instances of `function` and `})`? Eek! This is affectionately known as **callback hell**.

Writing better code isn't that hard! You only need to know about a few things:

# Name your functions

Here is some (messy) browser javascript that uses [browser-request](https://github.com/iriscouch/browser-request) to make an AJAX request to a server:

``` javascript
var form = document.querySelector('form')
form.onsubmit = function(submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, function(err, response, body) {
    var statusMessage = document.querySelector('.status')
    if (err) return statusMessage.value = err
    statusMessage.value = body
  })
}
```

This code has two anonymous functions. Let's give em names!

``` javascript
var form = document.querySelector('form')
form.onsubmit = function formSubmit(submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, function postResponse(err, response, body) {
    var statusMessage = document.querySelector('.status')
    if (err) return statusMessage.value = err
    statusMessage.value = body
  })
}
```

As you can see naming functions is super easy and does some nice things to your code:

- makes code easier to read
- when exceptions happen you will get stacktraces that reference actual function names instead of "anonymous"
- allows you to keep your code shallow, or not nested deeply, which brings me to my next point:


# Keep your code shallow

Building on the last example, let's go a bit further and get rid of the triple level nesting that is going on in the code:

``` javascript
function formSubmit(submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, postResponse)
}

function postResponse(err, response, body) {
  var statusMessage = document.querySelector('.status')
  if (err) return statusMessage.value = err
  statusMessage.value = body
}

document.querySelector('form').onsubmit = formSubmit
```

Code like this is less scary to look at and is easier to edit, refactor and hack on later.

# Modularize!

This is the most important part: **Anyone is capable of creating modules** (AKA libraries). To quote [Isaac Schlueter](http://twitter.com/izs) (of the node.js project): *"Write small modules that each do one thing, and assemble them into other modules that do a bigger thing. You can't get into callback hell if you don't go there."*

Let's take out the boilerplate code from above and turn it into a module by splitting it up into a couple of files. Since I write JavaScript in both the browser and on the server, I'll show a method that works in both but is still nice and simple.

Here is a new file called `formuploader.js` that contains our two functions from before:

``` javascript
function formSubmit(submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, postResponse)
}

function postResponse(err, response, body) {
  var statusMessage = document.querySelector('.status')
  if (err) return statusMessage.value = err
  statusMessage.value = body
}

exports.submit = formSubmit
```

The `exports` bit at the end is an example of the CommonJS module system, which is used by Node.js for server side javascript programming. I quite like this style of modules because it is so simple -- you only have to define what should be shared when the module gets required (that's what the `exports` thing is).

To use CommonJS modules in the browser you can use a command-line thing called [browserify](https://github.com/substack/node-browserify). I won't go into the details on how to use it here but it lets you use `require` to load modules into your programs.

Now that we have `formuploader.js` (and it is loaded in the page as a script tag) we just need to require it and use it! Here is how our application specific code looks now:

``` javascript
var formUploader = require('formuploader')
document.querySelector('form').onsubmit = formUploader.submit
```

Now our application is only two lines of code and has the following benefits:

- easier for new developers to understand -- they won't get bogged down by having to read through all of the `formuploader` functions
- `formuploader` can get used in other places without duplicating code and can easily be shared on github
- the code itself is nice and simple and easy to read

There are lots of module patterns for [web browser](http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth) and [on the server](http://nodejs.org/api/modules.html). Some of them get very complicated. The ones shown here are what I consider to be the simplest to understand.

# I still don't get it
Try reading my [introduction to callbacks](https://github.com/maxogden/art-of-node#callbacks).

# What about promises?
[Promises](http://domenic.me/2012/10/14/youre-missing-the-point-of-promises/) are a more abstract pattern of working with async code in JavaScript.

The scope of this document is to show how to write vanilla javascript. If you use a third party library that adds abstraction to your JS then make sure you're willing to force everyone that contributes to your library to also have the same views on JS as you.

In my own personal experience I use callbacks for 90% of the async code I write and when things get hairy I bring in something like the [async](https://github.com/caolan/async) library.

That being said, everyone develops their own unique JavaScript style and you should do what you like. Just remember that there are no absolutes: some people like to use only callbacks, some people don't.

# Additional reading

- [A pattern for decoupling DOM events from @dkastner](https://gist.github.com/3392235)

Please contribute new sections or fix existing ones by [forking this project on github](http://github.com/maxogden/callback-hell)!
