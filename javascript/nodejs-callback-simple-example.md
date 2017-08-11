# Node.js Callback Simple Example

From: [http://stackoverflow.com/questions/19739755/nodejs-callbacks-simple-example](http://stackoverflow.com/questions/19739755/nodejs-callbacks-simple-example)

```js
var myCallback = function(data) {
  console.log('got data: '+data);
};

var usingItNow = function(callback) {
  callback('get it?');
};
```



Now open node or browser console and paste the above definitions.

Finally use it with this next line:

```js
usingItNow(myCallback);
```

**With Respect to the Node-Style Error Conventions**

Costa asked what this would look like if we were to honor the node error callback conventions.

In this convention, the callback should expect to receive at least one argument, the first argument, as an error. Optionally we will have one or more additional arguments, depending on the context. In this case, the context is our above example.

Here I rewrite our example in this convention.

```js
var myCallback = function(err, data) {
  if (err) throw err; // Check for the error and throw if it exists.
  console.log('got data: '+data); // Otherwise proceed as usual.
};

var usingItNow = function(callback) {
  callback(null, 'get it?'); // I dont want to throw an error, so I pass null for the error argument
};
```

If we want to simulate an error case, we can define usingItNow like this

```js
var usingItNow = function(callback) {
  var myError = new Error('My custom error!');
  callback(myError, 'get it?'); // I send my error as the first argument.
};
```

The final usage is exactly the same as in above:

```js
usingItNow(myCallback);
```

The only difference in behavior would be contingent on which version of `usingItNow` you've defined: the one that feeds a "truthy value" \(an Error object\) to the callback for the first argument, or the one that feeds it null for the error argument.

