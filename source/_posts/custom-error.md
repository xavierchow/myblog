title: How to extend Javascript Error
tags:
  - javascript
date: 2016-06-02 23:34:10
---


# Motivation
When your application becomes larger and more complex, you may find the default Javascript `Error` is not enough, you might want to use different Errors for different modules or customize it with self-defined properties, then it's time to extend it properly.
<!-- more -->  

# How to extend error

## ES5
It's quite straightforward, using `util.inherits` and customize properties, that's all.
```js
var util = require('util');
function MyError(message) {
  this.name = 'MyError';
  this.message = message;
}

util.inherits(MyError, Error);

var err = new MyError('foo');
console.log(err);

```

## ES6
But it's 2016, we have node v4/v5/v6, you should use es6!
```js
class MyError extends Error {
  constructor(message) {
    this.name = 'MyError';
    this.message = message;
  }
}

const err = new MyError('bar');
console.log(err);
```
However the code above doesn't work, you will get an error `ReferenceError: this is not defined`,
the reason is forgetting to call `super()` in constructor, you can refer to [here](http://stackoverflow.com/questions/31067368/javascript-es6-class-extend-without-super) for details.
So is the following code good enough? 
```js
class MyError extends Error {
  constructor(message) {
    super();
    this.name = 'MyError';
    this.message = message;
  }
}

const err = new MyError('bar');
console.log(err);
```

## captureStackTrace
The answer is No if you care about call stack, consider the following code:
```js

class MyError extends Error {
  constructor(message) {
    super(); // (A)
    this.name = 'MyError';
    this.message = message;
  }
}

try {
  main();
} catch (e) {
  console.log(e.stack);
}
function main () {
  sub();
}
function sub () {
  throw new MyError('baz'); // (B)
}

```
You will find the stack frame is from (A) instead of (B), which in most of cases is not expected. The solution is using [captureStackTrace](https://nodejs.org/api/errors.html#errors_error_capturestacktrace_targetobject_constructoropt). The defination is as follows.
```
Error.captureStackTrace(targetObject[, constructorOpt])
```
It mainly creates the `.stack` property for the error which indicates the location where the error is from. The first param is an object which relates to the first line of stack, but just note this sentence in document `The first line of the trace, instead of being prefixed with ErrorType: message, will be the result of calling targetObject.toString()` , actually it's not ture, the `toString` doesn't work at least for node v4, there's also a github [issue](https://github.com/nodejs/node/issues/5675) for it, it just concatenates the `name` and `message` together for now, let's skip it as it's not so important.

The second param is what we need to hide the stack frame, given a function, all frame above it(including itself) will be omit from the stack.
```js

class MyError extends Error {
  constructor(message) {
    super(); // (A)
    this.name = 'MyError';
    this.message = message;
    Error.captureStackTrace(this, MyError); // added
  }
}

try {
  main();
} catch (e) {
  console.log(e.stack);
}
function main () {
  sub();
}
function sub () {
  throw new MyError('baz'); // (B)
}

```
This time the stack frame is from (B). Quite simple, isn't it? 

# Ending
There's still something to be imporved in fact, e.g compatibility and robustness, but I will not elaborate here, 
because there's already a tiny but convenient [boilerplate](https://github.com/bjyoungblood/es6-error) you can leverage, give it a try!

