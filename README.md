# Understanding "this" in javascript with regular functions
## Conceptual Overview of "this"
When a function is created, a keyword called ` this ` is created (behind the scenes), which links to the object in which the function operates.

**The `this` keyword’s value has nothing to do with the function itself, how the function is called determines this's value **

## Default "this" context
```javascript
// define a function
var myFunction = function () {
  console.log(this);
};

// call it
myFunction();
```

What can we expect the `this` value to be? By default, `this` should always be the window Object, which refers to the root—the global scope, except if the script is running in strict mode (`"use strict"`) `this` will be undefined.

## Object literals
```javascript
var myObject = {
  myMethod: function () {
    console.log(this);
  }
};
```
What would be the `this` context here?...

* this === myObject?
* this === window?
* this === anything else?

Well, the answer is We do not know.

Remember

The this keyword’s value has nothing to do with the function itself, how the function is called determines the `this` value

Okay, let's change the code a bit...
```javascript
var myMethod = function () {
  console.log(this);
};

var myObject = {
  myMethod: myMethod
};
```
Is it clearer now?
Of course, everything depends on how we call the function.

myObject in the code is given a property called myMethod, which points to the myMethod function. When the myMethod function is called from the global scope, this refers to the window object. When it is called as a method of myObject, this refers to myObject.
```javascript
myObject.myMethod() // this === myObject
myMethod() // this === window
```
This is called implicit binding

## Explicit binding
Explicit binding is when we explicitly bind a context to the function. This is done with call() or apply()
```javascript
var myMethod = function () {
  console.log(this);
};

var myObject = {
  myMethod: myMethod
};

myMethod() // this === window
myMethod.call(myObject, args1, args2, ...) // this === myObject
myMethod.apply(myObject, [array of args]) // this === myObject
```

Which is more precedent, implicit binding or explicit binding?

```javascript
var myMethod = function () { 
  console.log(this.a);
};

var obj1 = {
  a: 2,
  myMethod: myMethod
};

var obj2 = {
  a: 3,
  myMethod: myMethod
};

obj1.myMethod(); // 2
obj2.myMethod(); // 3

obj1.myMethod.call( obj2 ); // ?????
obj2.myMethod.call( obj1 ); // ?????
```
Explicit binding takes precedence over implicit binding, which means you should ask first if explicit binding applies before checking for implicit binding.
```javascript
obj1.myMethod.call( obj2 ); // 3
obj2.myMethod.call( obj1 ); // 2
```
## Hard binding
This is done with bind() (ES5). bind() returns a new function that is hard-coded to call the original function with the this context set as you specified.
```javascript
myMethod = myMethod.bind(myObject);

myMethod(); // this === myObject
```
Hard binding takes precedence over explicit binding.
```javascript
var myMethod = function () { 
  console.log(this.a);
};

var obj1 = {
  a: 2
};

var obj2 = {
  a: 3
};

myMethod = myMethod.bind(obj1); // 2
myMethod.call( obj2 ); // 2
```
## New binding
```javascript
function foo(a) {
  this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2

```
So this when the function has been called with new refers to the new instance created.

When a function is called with new, it does not care about implicit, explicit, or hard binding, it just creates the new context—which is the new instance.
```javascript
function foo(something) {
  this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```
## API calls
Sometimes, we use a library or a helper object which does something (Ajax, event handling, etc.) and it calls a passed callback. Well, we have to be careful in these cases. Example:
```javascript
myObject = {
  myMethod: function () {
    helperObject.doSomethingCool('superCool',
      this.onSomethingCoolDone);
    },

    onSomethingCoolDone: function () {
      /// Only god knows what is "this" here
    }
};
```
Take a look at the code. You might think that because we are passing this.onSomethingCoolDone as a callback, the scope is myObject a reference to that method and not to the way to call it.

To fix this, there are a few ways:

Usually libraries offer another parameter, so then you can pass the scope you want to get back.
```javascript
myObject = {
  myMethod: function () {
    helperObject.doSomethingCool('superCool', this.onSomethingCoolDone, this);
  },

  onSomethingCoolDone: function () {
    /// Now everybody know that "this" === myObject
  }
};
```
You can hard bind the method to the scope you want (ES5).
```javascript
myObject = {
  myMethod: function () {
    helperObject.doSomethingCool('superCool', this.onSomethingCoolDone.bind(this));
  },

  onSomethingCoolDone: function () {
    /// Now everybody know that "this" === myObject
  }
};
```
You can create a closure and cache this into me. For example:
```javascript
myObject = {
  myMethod: function () {
    var me = this;

    helperObject.doSomethingCool('superCool', function () {
      /// Only god knows what is "this" here, but we have access to "me"
    });
  }
};
```
I do not recommend this approach because it can cause memory leaks and it tends to make you forget about the real scope and rely on variables. You can get to the point where your scope is a real mess.

# Understanding "this" in javascript with arrow functions

We will go through the same examples, but we will use arrow functions instead to compare the outputs.

The motivation of this second post about the scope is, despite arrow functions are a powerful addition to ES6, they must not be misused or abused.

## Default "this" context
Arrow functions do not bind their own this, instead, they inherit the one from the parent scope, which is called "lexical scoping". This makes arrow functions to be a great choice in some scenarios but a very bad one in others

If we look at the first example but using arrow functions

```javascript
// define a function
const myFunction = () => {
  console.log(this);
};
```
// call it
myFunction();
What can we expect this to be?.... exactly, same as with normal functions, window or global object. Same result but not the same reason. With normal functions the scoped is bound to the global one by default, arrows functions, as I said before, do not have their own this but they inherit it from the parent scope, in this case the global one.

What would happen if we add "use strict"? Nothing, it will be the same result, since the scope comes from the parent one.

## Arrow functions as methods
```javascript
const myObject = {
  myMethod: () => {
    console.log(this);
  }
};
```
What about now?

In this case, one could say, that it really depends on how the method is called, same as normal functions, but that's not the case here, let's see...
```javascript
myObject.myMethod() // this === window or global object

const myMethod = myObject.myMethod;
myMethod() // this === window or global object
```
Weird right? Well, remember, arrow functions don't bind their own scope, but inherit it from the parent one, which in this case is window or the global object.

Let's change the example a little bit
```javascript
const myObject = {
  myArrowFunction: null,
  myMethod: function () {
    this.myArrowFunction = () => { console.log(this) };
  }
};
```
We need to call myObject.myMethod() to initialize myObject.myArrowFunction and then let's see what the output would be
```javascript
myObject.myMethod() // this === myObject

myObject.myArrowFunction() // this === myObject
`
const myArrowFunction = myObject.myArrowFunction;
myArrowFunction() // this === myObject
```
Clearer now? When we call `myObject.myMethod()`, we initialize myObject.myArrowFunction with an arrow function which is inside of the method myMethod, so it will inherit its scope. We can clearly see a perfect use case, closures.

## Explicit, Hard and New binding
What would happen when we try to bind a scope with any of these techniques?

let's see...
```javascript
const myMethod = () => {
  console.log(this);
};

const myObject = {};
```
## Explicity binding
```javascript
myMethod.call(myObject, args1, args2, ...) // this === window or global object
myMethod.apply(myObject, [array of args]) // this === window or global object
```
## Hard binding
```javascript
const myMethodBound = myMethod.bind(myObject);

myMethodBound(); // this === window or global object
```
## New binding
new myMethod(); // Uncaught TypeError: myMethod is not a constructor
As you see, it does not matter how we try to bind the scope, it will never work. Also, arrows functions are not constructors so you can not use new with them.

## API calls
This part is interesting. Arrow functions are a good choice for API calls ( asynchronous code ), only if we use CLOSURES, let's look at this...

```javascript
myObject = {
  myMethod: function () {
    helperObject.doSomethingAsync('superCool', () => {
      console.log(this); // this === myObject
    });
  },
};
```
This is the perfect example, we ask to do something async, we wait for the answer to do some actions and we don't have to worry about the scope we were working with.

But what would happen if for any reason we refactor the code and extract that function out in order to be reused, for example?

let's see...

```javascript
const reusabledCallback = () => {
  console.log(this); // this === window or global object
};

myObject = {
  myMethod: function () {
    helperObject.doSomethingAsync('superCool', reusabledCallback);
  },
};
```
If we do so, we would be breaking the current working code, and, remember, it doesn't matter how we try to bind the scope, it won't work. So if you decide to do so, you have to use normal functions and bind the scope manually. For example

const reusabledCallback = function () {
  console.log(this);
};

```javascript
myObject = {
  myMethod: function () {
    helperObject.doSomethingAsync('superCool', reusabledCallback.bind(myObject));
  },
};

```
## Conclusions
Arrow functions are a powerful addition to ES6, but we have to be careful and wise when and how to use them. I continuously find places, where arrow functions are not usable, and this can cause difficult to track errors, especially if we do not understand how they really work.

In my opinion, arrow functions are the best choice when working with closures or callbacks, but not a good choice when working with class/object methods or constructors.
