From [http://stackabuse.com/avoiding-callback-hell-in-node-js/](http://stackabuse.com/avoiding-callback-hell-in-node-js/)

## Avoiding Callback Hell in Node.js

## Introduction {#introduction}

I'll admit that I was one of those people that decided to learn Node.js simply because of the buzz around it and how much everyone was talking about it. I figured there must be something special about it if it has this much support so early on in its life. I mostly came from a C, Java, and Python background, so JavaScript's asynchronous style was much different than anything I had encountered before.

As many of you probably know, all JavaScript really is underneath is a single-threaded event loop that processes queued events. If you were to execute a long-running task within a single thread then the process would block, causing other events to have to wait to be processed \(i.e. UI hangs, data doesn't get saved, etc\). This is exactly what you want to avoid in an event-driven system.[Here](https://www.youtube.com/watch?v=QgwSUtYSUqA)is a great video explaining much more about the JavaScript event loop.

To solve this blocking problem, JavaScript heavily relies on callbacks, which are functions that run after a long-running process \(IO, timer, etc\) has finished, thus allowing the code execution to proceed past the long-running task.

```js
downloadFile('example.com/weather.json', function(err, data) {  
    console.log('Got weather data:', data);
});
```

## The problem: Callback hell {#theproblemcallbackhell}

While the concept of callbacks is great in theory, it can lead to some really confusing and difficult-to-read code. Just imagine if you need to make callback after callback:

```js
getData(function(a){  
    getMoreData(a, function(b){
        getMoreData(b, function(c){ 
            getMoreData(c, function(d){ 
                getMoreData(d, function(e){ 
                    ...
                });
            });
        });
    });
});
```

As you can see, this can really get out of hand. Throw in some `if`  statements, `for` loops, function calls, or comments and you'll have some very hard-to-read code. Beginners especially fall victim to this, not understanding how to avoid this "pyramid of doom".

## Alternatives {#alternatives}

### Design around it {#designaroundit}

So many programmers get caught up in callback hell because of this \(poor design\) alone. They don't really think about their code structure ahead of time and don't realize how bad their code has gotten until after its too late. As with any code you're writing, you should stop and think about what can be done to make it simpler and more readable before, or while, writing it. Here are a few tips you can use to**avoid callback hell**\(or at least manage it\).

#### Use modules {#usemodules}

In just about every programming language, one of the best ways to reduce complexity is to modularize. JavaScript programming is no different. Whenever you're writing code, take some time to step back and figure out if there has been a common pattern you frequently encounter.

Are you writing the same code multiple times in different places? Do different parts of your code follow a common theme? If so, you have an opportunity to clean things up and abstract out and reuse code.

There are thousands of modules out there you can look at for reference, but here are a few to consider. They handle common, but very specific, tasks that would otherwise clutter your code and reduce readability:[Pluralize](https://github.com/blakeembrey/pluralize),[csv](https://www.npmjs.com/package/csv),[qs](https://www.npmjs.com/package/qs),[clone](https://www.npmjs.com/package/clone).

#### Give your functions names {#giveyourfunctionsnames}

When reading code \(especially messy, unorganized code\), its easy to lose track of the logic flow, or even syntax, when small spaces are congested with so many nested callbacks. One way to help combat this is to name your functions, so all you'll have to do is glance at the name and you'll a better idea as to what it does. It also gives your eyes a syntax reference point.

Consider the following code:

```js
var fs = require('fs');

var myFile = '/tmp/test';  
fs.readFile(myFile, 'utf8', function(err, txt) {  
    if (err) return console.log(err);

    txt = txt + '\nAppended something!';
    fs.writeFile(myFile, txt, function(err) {
        if(err) return console.log(err);
        console.log('Appended text!');
    });
});
```

Looking at this may take you a few seconds to realize what each callback does and where it starts. Adding a little extra information \(names\) to the functions can make a big difference for readability, especially when you're multiple levels deep in callbacks:

```js
var fs = require('fs');

var myFile = '/tmp/test';  
fs.readFile(myFile, 'utf8', function appendText(err, txt) {  
    if (err) return console.log(err);

    txt = txt + '\nAppended something!';
    fs.writeFile(myFile, txt, function notifyUser(err) {
        if(err) return console.log(err);
        console.log('Appended text!');
    });
});
```

Now just a quick glance will tell you the first function appends some text while the second function notifies the user of the change.

#### Declare your functions beforehand {#declareyourfunctionsbeforehand}

One of the best ways to reduce code clutter is by maintaining better separation of code. If you declare a callback function beforehand and call it later, you'll avoid the deeply nested structures that make callback hell so difficult to work with.

So you could go from this...

```js
var fs = require('fs');

var myFile = '/tmp/test';  
fs.readFile(myFile, 'utf8', function appendText(err, txt) {  
    if (err) return console.log(err);

    txt = txt + '\nAppended something!';
    fs.writeFile(myFile, txt, function notifyUser(err) {
        if(err) return console.log(err);
        console.log('Appended text!');
    });
});
```

...to this:

```js
var fs = require('fs');

var myFile = '/tmp/test';  
fs.readFile(myFile, 'utf8', function(err, txt) {  
    if (err) return console.log(err);

    txt = txt + '\nAppended something!';
    fs.writeFile(myFile, txt, function(err) {
        if(err) return console.log(err);
        console.log('Appended text!');
    });
});
```

While this can be a great way to help ease the problem, it doesn't completely solve the problem. When reading code written in this way, if you don't remember exactly what each function does then you'll have to go back and look at each one to retrace the logic flow, which can take time.

### Async.js {#asyncjs}

Thankfully, libraries like[Async.js](https://github.com/caolan/async)exist to try and curb the problem. Async adds a thin layer of functions on top of your code, but can greatly reduce the complexity by avoiding callback nesting.

Many helper methods exist in Async that can be used in different situations, like[series](https://github.com/caolan/async#seriestasks-callback),[parallel](https://github.com/caolan/async#parallel),[waterfall](https://github.com/caolan/async#waterfall), etc. Each function has a specific use-case, so take some time to learn which one will help in which situations.

As good as Async is, like anything, its not perfect. Its very easy to get carried away by combining series, parallel, forever, etc, at which point you're right back to where you started with messy code. Be careful not to prematurely optimize. Just because a few async tasks can be run in parallel doesn't always mean they should. In reality, since Node is only single-threaded, running tasks in parallel on using Async has little to no performance gain.

The code from above can be simplified using Async's waterfall:

```js
var fs = require('fs');  
var async = require('async');

var myFile = '/tmp/test';

async.waterfall([  
    function(callback) {
        fs.readFile(myFile, 'utf8', callback);
    },
    function(txt, callback) {
        txt = txt + '\nAppended something!';
        fs.writeFile(myFile, txt, callback);
    }
], function (err, result) {
    if(err) return console.log(err);
    console.log('Appended text!');
});
```

### Promises {#promises}

Although Promises can take a bit to grasp, in my opinion they are one of the more important concepts you can learn in JavaScript. During development of one of my[SaaS apps](https://polymetrics.io/), I ended up rewriting the entire codebase using Promises. Not only did it reduce the number of lines of code drastically, but it made the logical flow of the code much easier to follow.

Here is an example using the very fast and very popular Promise library,[Bluebird](https://github.com/petkaantonov/bluebird):

```js
var Promise = require('bluebird');  
var fs = require('fs');  
Promise.promisifyAll(fs);

var myFile = '/tmp/test';  
fs.readFileAsync(myFile, 'utf8').then(function(txt) {  
    txt = txt + '\nAppended something!';
    fs.writeFile(myFile, txt);
}).then(function() {
    console.log('Appended text!');
}).catch(function(err) {
    console.log(err);
});
```

Notice how this solution is not only shorter than the previous solutions, but it is easier to read as well \(although, admittedly, Promise-style code can take some getting used to\). Take the time to learn and understand Promises, it'll be worth your time. However, Promises are definitely not the solution to all our problems in asynchronous programming, so don't assume by using them you will have a fast, clean, bug-free app. The key is knowing when they'll be useful to you.

A few Promise libraries you should check out are[Q](https://github.com/kriskowal/q),[Bluebird](https://github.com/petkaantonov/bluebird), or the[built-in Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)if you're using ES6.

### Async/Await {#asyncawait}

_Note: This is an ES7 feature, which currently isn't supported in Node or io.js. However, you can use it right now with a transpiler like_[_Babel_](https://babeljs.io/)_._

Another option to clean up your code, and my soon-to-be favorite \(when it has broader support\), is using`async`functions. This will allow you to write code that looks much more like synchronous code, yet is still asynchronous.

An example:

```js
async function getUser(id) {  
    if (id) {
        return await db.user.byId(id);
    } else {
        throw 'Invalid ID!';
    }
}

try {  
    let user = await getUser(123);
} catch(err) {
    console.error(err);
}
```

The `db.user.byId(id)` call returns a `Promise`, which we'd normally have to use with`.then()`, but with `await` we can return the resolved value directly.

Notice that the function containing the `await` call is prefixed with `async`, which tells us that it contains asynchronous code and must also be called with `await`.

Another big advantage to this method is we can now use `try/catch`, `for`, and `while` with our asynchronous functions, which is much more intuitive than chaining promises together.

Aside from using transpilers like Babel and[Traceur](https://github.com/google/traceur-compiler), you can also get functionality like this in Node with the[asyncawait](https://www.npmjs.com/package/asyncawait)package.

## Conclusion {#conclusion}

Avoid such common problems as callback hell isn't easy, so don't expect to end your frustrations right away. We all get caught in it. Just try to slow down and take some time to think about the structure of your code. Like anything, practice makes perfect.

_Have you run in to callback hell? If so, how do you get around it? Tell us in the comments!_

  


