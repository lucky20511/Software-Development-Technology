From: https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Inheritance

## Prototypal inheritance {#Prototypal_inheritance}

So far we have seen some inheritance in action — we have seen how prototype chains work, and how members are inherited going up a chain. But mostly this has involved built-in browser functions. How do we create an object in JavaScript that inherits from another object?

As mentioned earlier in the course, some people think JavaScript is not a true object-oriented language. In "classic OO" languages, you tend to define class objects of some kind, and you can then simply define which classes inherit from which other classes \(see[C++ inheritance](http://www.tutorialspoint.com/cplusplus/cpp_inheritance.htm)for some simple examples\). JavaScript uses a different system — "inheriting" objects do not have functionality copied over to them, instead the functionality they inherit is linked to via the prototype chain \(often referred to as**prototypal inheritance**\).

Let's explore how to do this with a concrete example.

## Getting started {#Getting_started}

First of all, make yourself a local copy of our [oojs-class-inheritance-start.htm l](https://github.com/mdn/learning-area/blob/master/javascript/oojs/advanced/oojs-class-inheritance-start.html)file \(see it [running live](http://mdn.github.io/learning-area/javascript/oojs/advanced/oojs-class-inheritance-start.html) also\). Inside here you'll find the same`Person()`constructor example that we've been using all the way through the module, with a slight difference — we've defined only the properties inside the constructor:

```
function Person(first, last, age, gender, interests) {
  this.name = {
    first,
    last
  };
  this.age = age;
  this.gender = gender;
  this.interests = interests;
};
```

The methods are_all_defined on the constructor's prototype. For example:

```
Person.prototype.greeting = function() {
  alert('Hi! I\'m ' + this.name.first + '.');
};
```

**Note**: In the source code, you'll also see`bio()`and`farewell()`methods defined. Later you'll see how these can be inherited by other constructors.

Say we wanted to create a`Teacher`class, like the one we described in our initial object-oriented definition, which inherits all the members from`Person`, but also includes:

1. A new property,
   `subject`
   — this will contain the subject the teacher teaches.
2. An updated
   `greeting()`
   method, which sounds a bit more formal than the standard
   `greeting()`
   method — more suitable for a teacher addressing some students at school.

## Defining a Teacher\(\) constructor function {#Defining_a_Teacher()_constructor_function}

The first thing we need to do is create a`Teacher()`constructor — add the following below the existing code:

```
function Teacher(first, last, age, gender, interests, subject) {
  Person.call(this, first, last, age, gender, interests);

  this.subject = subject;
}
```

This looks similar to the Person constructor in many ways, but there is something strange here that we've not seen before — the[`call()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)function. This function basically allows you to call a function defined somewhere else, but in the current context. The first parameter specifies the value of`this`that you want to use when running the function, and the other parameters are those that should be passed to the function when it is invoked.

We want the`Teacher()`constructor to take the same parameters as the`Person()`constructor it is inheriting from, so we specify them all as parameters in the`call()`invocation.

The last line inside the constructor simply defines the new`subject`property that teachers are going to have, which generic people don't have.

As a note, we could have simply done this:

```
function Teacher(first, last, age, gender, interests, subject) {
  this.name = {
    first,
    last
  };
  this.age = age;
  this.gender = gender;
  this.interests = interests;
  this.subject = subject;
}
```

But this is just redefining the properties anew, not inheriting them from`Person()`, so it defeats the point of what we are trying to do. It also takes more lines of code.

### Inheriting from a constructor with no parameters {#Inheriting_from_a_constructor_with_no_parameters}

Note that if the constructor you are inheriting from doesn't take its property values from parameters, you don't need to specify them as additional arguments in`call()`. So, for example,  if you had something really simple like this:

```
function Brick() {
  this.width = 10;
  this.height = 20;
}
```

You could inherit the`width`and`height`properties by doing this \(as well as the other steps described below, of course\):

```
function BlueGlassBrick() {
  Brick.call(this);

  this.opacity = 0.5;
  this.color = 'blue';
}
```

Note that we've only specified`this`inside`call()`— no other parameters are required as we are not inheriting any parameters, only properties.

## Setting Teacher\(\)'s prototype and constructor reference {#Setting_Teacher()'s_prototype_and_constructor_reference}

All is good so far, but we have a problem. We have defined a new constructor, and it has a`prototype`property, which by default just contains a reference to the constructor function itself. It does not contain the methods of the Person constructor's`prototype`property. To see this, enter`Object.getOwnPropertyNames(Teacher.prototype)` into either the text input field or your JavaScript console. Then enter it again, replacing`Teacher`with`Person`. Nor does the new constructor_inherit_those methods. To see this, compare the outputs of`Person.prototype.greeting`and`Teacher.prototype.greeting`. We need to get`Teacher()`to inherit the methods defined on`Person()`'s prototype. So how do we do that?

1. Add the following line below your previous addition:
   ```
   Teacher.prototype = Object.create(Person.prototype);
   ```

   Here our friend
   [`create()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
   comes to the rescue again. In this case we are using it to create a new object and make it the value of`Teacher.prototype`. The new object has `Person.prototype`as its prototype and will therefore inherit, if and when needed, all the methods available on`Person.prototype`.
2. We need to do one more thing before we move on. After adding the last line, `Teacher.prototype`'s`constructor`property is now equal to`Person()`, because we just set`Teacher.prototype`to reference an object that inherits its properties from `Person.prototype`! Try saving your code, loading the page in a browser, and entering`Teacher.prototype.constructor`into the console to verify.
3. This can become a problem, so we need to set this right. You can do so by going back to your source code and adding the following line at the bottom:
   ```
   Teacher.prototype.constructor = Teacher;
   ```
4. Now if you save and refresh, entering`Teacher.prototype.constructor`should return`Teacher()`, as desired, plus we are now inheriting from`Person()`!

## Giving Teacher\(\) a new greeting\(\) function {#Giving_Teacher()_a_new_greeting()_function}

To finish off our code, we need to define a new`greeting()`function on the`Teacher()`constructor.

The easiest way to do this is to define it on`Teacher()`'s prototype — add the following at the bottom of your code:

```
Teacher.prototype.greeting = function() {
  var prefix;

  if (this.gender === 'male' || this.gender === 'Male' || this.gender === 'm' || this.gender === 'M') {
    prefix = 'Mr.';
  } else if (this.gender === 'female' || this.gender === 'Female' || this.gender === 'f' || this.gender === 'F') {
    prefix = 'Mrs.';
  } else {
    prefix = 'Mx.';
  }

  alert('Hello. My name is ' + prefix + ' ' + this.name.last + ', and I teach ' + this.subject + '.');
};
```

This alerts the teacher's greeting, which also uses an appropriate name prefix for their gender, worked out using a conditional statement.

## Trying the example out {#Trying_the_example_out}

Now that you've entered all the code, try creating an object instance from`Teacher()`by putting the following at the bottom of your JavaScript \(or something similar of your choosing\):

```
var teacher1 = new Teacher('Dave', 'Griffiths', 31, 'male', ['football', 'cookery'], 'mathematics');
```

Now save and refresh, and try accessing the properties and methods of your new`teacher1`object, for example:

```
teacher1.name.first;
teacher1.interests[0];
teacher1.bio();
teacher1.subject;
teacher1.greeting();
teacher1.farewell();
```

These should all work just fine. The queries on lines 1, 2, 3, and 6 access members inherited from the generic`Person()`constructor \(class\). The query on line 4 accesses a member that is available only on the more specialized`Teacher()`constructor \(class\). The query on line 5 would have accessed a member inherited from`Person()`, except for the fact that`Teacher()`has its own member with the same name, so the query accesses that member.

**Note**: If you have trouble getting this to work, compare your code to our [finished version](https://github.com/mdn/learning-area/blob/master/javascript/oojs/advanced/oojs-class-inheritance-finished.html)\(see it [running live](http://mdn.github.io/learning-area/javascript/oojs/advanced/oojs-class-inheritance-finished.html) also\).

The technique we covered here is not the only way to create inheriting classes in JavaScript, but it works OK, and it gives you a good idea about how to implement inheritance in JavaScript.

You might also be interested in checking out some of the new [ECMAScript](https://developer.mozilla.org/en-US/docs/Glossary/ECMAScript) features that allow us to do inheritance more cleanly in JavaScript \(see [Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)\). We didn't cover those here, as they are not yet supported very widely across browsers. All the other code constructs we discussed in this set of articles are supported as far back as IE9 or earlier, and there are ways to achieve earlier support than that.

A common way is to use a JavaScript library — most of the popular options have an easy set of functionality available for doing inheritance more easily and quickly. [CoffeeScript](http://coffeescript.org/#classes) for example provides`class`,`extends`, etc.

## A further exercise {#A_further_exercise}

In our [OOP theory section](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object-oriented_JS#Object-oriented_programming_from_10000_meters), we also included a`Student`class as a concept, which inherits all the features of`Person`, and also has a different`greeting()`method from `Person`that is much more informal than the`Teacher`'s greeting. Have a look at what the student's greeting looks like in that section, and try implementing your own`Student()`constructor that inherits all the features of`Person()`, and implements the different`greeting()`function.

**Note**: If you have trouble getting this to work, have a look at our[finished version](https://github.com/mdn/learning-area/blob/master/javascript/oojs/advanced/oojs-class-inheritance-student.html)\(see it[running live](http://mdn.github.io/learning-area/javascript/oojs/advanced/oojs-class-inheritance-student.html)also\).

## Object member summary {#Object_member_summary}

To summarize, you've basically got three types of property/method to worry about:

1. Those defined inside a constructor function that are given to object instances. These are fairly easy to spot — in your own custom code, they are the members defined inside a constructor using the`this.x = x`type lines; in built in browser code, they are the members only available to object instances \(usually created by calling a constructor using the`new`
   keyword, e.g.`var myInstance = new myConstructor()`\).
2. Those defined directly on the constructor themselves, that are available only on the constructor. These are commonly only available on built-in browser objects, and are recognized by being chained directly onto a constructor, _not _an instance. For example,[`Object.keys()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys).
3. Those defined on a constructor's prototype, which are inherited by all instances and inheriting object classes. These include any member defined on a Constructor's prototype property, e.g.`myConstructor.prototype.x()`.

If you are not sure which is which, don't worry about it just yet — you are still learning, and familiarity will come with practice.

## When would you use inheritance in JavaScript? {#When_would_you_use_inheritance_in_JavaScript}

Particularly after this last article, you might be thinking "woo, this is complicated". Well, you are right. Prototypes and inheritance represent some of the most complex aspects of JavaScript, but a lot of JavaScript's power and flexibility comes from its object structure and inheritance, and it is worth understanding how it works.

In a way, you use inheritance all the time. Whenever you use various features of a Web API , or methods/properties defined on a built-in browser object that you call on your strings, arrays, etc., you are implicitly using inheritance.

In terms of using inheritance in your own code, you probably won't use it often, especially to begin with, and in small projects. It is a waste of time to use objects and inheritance just for the sake of it when you don't need them. But as your code bases get larger, you are more likely to find a need for it. If you find yourself starting to create a number of objects that have similar features, then creating a generic object type to contain all the shared functionality and inheriting those features in more specialized object types can be convenient and useful.

**Note**: Because of the way JavaScript works, with the prototype chain, etc., the sharing of functionality between objects is often called **delegation**. Specialized objects delegate functionality to a generic object type. This is probably more accurate than calling it_inheritance_, as the "inherited" functionality is not copied to the objects that are doing the "inheriting". Instead it still remains in the generic object.

When using inheritance, you are advised to not have too many levels of inheritance, and to keep careful track of where you define your methods and properties. It is possible to start writing code that temporarily modifies the prototypes of built-in browser objects, but you should not do this unless you have a really good reason. Too much inheritance can lead to endless confusion, and endless pain when you try to debug such code.

Ultimately, objects are just another form of code reuse, like functions or loops, with their own specific roles and advantages. If you find yourself creating a bunch of related variables and functions and want to track them all together and package them neatly, an object is a good idea. Objects are also very useful when you want to pass a collection of data from one place to another. Both of these things can be achieved without use of constructors or inheritance. If you only need a single instance of an object, then you are probably better off just using an object literal, and you certainly don't need inheritance.

## Summary {#Summary}

This article has covered the remainder of the core OOJS theory and syntax that we think you should know now. At this point you should understand JavaScript object and OOP basics, prototypes and prototypal inheritance, how to create classes \(constructors\) and object instances, add features to classes, and create subclasses that inherit from other classes.

