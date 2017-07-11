# Understand JavaScript Closures With Ease

From: [http://javascriptissexy.com/understand-javascript-closures-with-ease/](http://javascriptissexy.com/understand-javascript-closures-with-ease/)

### What is a closure?

A closure is an inner function that has access to the outer \(enclosing\) function’s variables—scope chain. The closure has three scope chains: it has access to its own scope \(variables defined between its curly brackets\), it has access to the outer function’s variables, and it has access to the global variables.

The inner function has access not only to the outer function’s variables, but also to the outer function’s parameters. Note that the inner function cannot call the outer function’s arguments object, however, even though it can call the outer function’s parameters directly.

You create a closure by adding a function inside another function.

### A Basic Example of Closures in JavaScript:

```
1
```

Closures are used extensively in Node.js; they are workhorses in Node.js’ asynchronous, non-blocking architecture. Closures are also frequently used in jQuery and just about every piece of JavaScript code you read.

A Classic jQuery Example of Closures:

```
1
```

### Closures’ Rules and Side Effects

Closures have access to the outer function’s variable even after the outer function returns:

One of the most important and ticklish features with closures is that the inner function still has access to the outer function’s variables even after the outer function has returned. Yep, you read that correctly. When functions in JavaScript execute, they use the same scope chain that was in effect when they were created. This means that even after the outer function has returned, the inner function still has access to the outer function’s variables. Therefore, you can call the inner function later in your program. This example demonstrates:

```
1
```

Closures store references to the outer function’s variables; they do not store the actual value.  Closures get more interesting when the value of the outer function’s variable changes before the closure is called. And this powerful feature can be harnessed in creative ways, such as this private variables example first demonstrated by Douglas Crockford: 





