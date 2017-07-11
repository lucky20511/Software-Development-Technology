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

Closures Gone Away

Because closures have access to the updated values of the outer function’s variables, they can also lead to bugs when the outer function’s variable changes with a for loop. Thus:

```
1
```

In the preceding example, by the time the anonymous functions are called, the value of i is 3 \(the length of the array and then it increments\). The number 3 was added to the uniqueID to create 103 for ALL the celebritiesID. So every position in the returned array get id = 103, instead of the intended 100, 101, 102.

The reason this happened was because, as we have discussed in the previous example, the closure \(the anonymous function in this example\) has access to the outer function’s variables by reference, not by value. So just as the previous example showed that we can access the updated variable with the closure, this example similarly accessed the i variable when it was changed, since the outer function runs the entire for loop and returns the last value of i, which is 103.

To fix this side effect \(bug\) in closures, you can use an Immediately Invoked Function Expression \(IIFE\), such as the following:

function celebrityIDCreator \(theCelebrities\) {

    var i;

    var uniqueID = 100;

    for \(i = 0; i &lt; theCelebrities.length; i++\) {

        theCelebrities\[i\]\["id"\] = function \(j\)  {

            return function \(\) {

                return uniqueID + j;

            } \(\) 

        } \(i\); 

    }

​

    return theCelebrities;

}   

​

​var actionCelebs = \[{name:"Stallone", id:0}, {name:"Cruise", id:0}, {name:"Willis", id:0}\];

​

var createIdForActionCelebs = celebrityIDCreator \(actionCelebs\);



​var stalloneID = createIdForActionCelebs \[0\];

 console.log\(stalloneID.id\); 

​

​var cruiseID = createIdForActionCelebs \[1\]; console.log\(cruiseID.id\);



