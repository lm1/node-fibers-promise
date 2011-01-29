# Intoruction

`fibers-promise` is a small yet powerful library based on [fibers](https://github.com/laverdet/node-fibers).

A promise is just a value container with build-in synchronisation mechanism. Promises allow you to get rid of callbacks clutter.

You may also like [fiberize](http://github.com/lm1/node-fiberize).

# Getting started

    npm install fibers-promise
    
This will install node-fibers as well. (Working g++ and node headers are required.)

Then run your code with:

    node-fibers your_file.js

# Example

To fetch an url:

    var http = require('http');
    var promise = require('fibers-promise');
    
    promise.start(function() {
      
      var p = promise();
    
      http.get({
          host: 'www.google.com',
          port: 80,
          path: '/'
      }, p);
    
      var res = p.get();
      res.setEncoding('utf8');
      res.on('data', p);
      res.on('end', p);
    
      var data, chunk;
      while (chunk = p.get()) {
        data += chunk;
      }
      console.log(data);
    });

# Interface

    var promise = require('fibers-promise');

### promise()

`promise` is a factory method returning new `Promise` object with the following methods:

  - `Promise.set(value)`

  `set` method sets the promise value and resumes a fiber waiting for it (if any).

  - `Promise(...)`

  `Promise` object is itself a function which when called sets promise value like a `set` method.
  If more than a single value is provided, promise value is set to an array containing all provided values.
  Thus it's possible to pass promise just as a callback to the asynchronous function.

  - `Promise.get()`

  Waits until the promise has a value and returns the value.

  - `Promise.ready()`

  Checks if promise has a value already. Returns a boolean.

  - `Promise.wait()`

  Waits until a promise has a value.

### promise.t()

This is also a factory method similar to `promise()`, but the `get()` method of this promise behaves as follows:

  - Single argument has been provided to `Promise(...)`

    If the value evaluates to true it's thrown; nothing is returned.

  - Two arguments were provided to `Promise(...)`

    If the first value evaluates to true it's thrown, second value is returned.

  - Three or more arguments were provided to `Promise(...)`

    If the first value evaluates to true it's thrown; remaining values are returned in form of an Array.

This type of promise is useful when passed as callback to most node asynchronous methods.

### promise.start(f, [args...])

`start` runs `f` in a new fiber and passes `args` to `f`.

### promise.task(f)

Returns a function which will execute `f` in new fiber upon invocation. All the arguments of the returned function are passed directly to `f`. Useful to postpone start of a fiber, or wrap a callback with a fiber.

### promise.waitAny(...)

Waits for multiple promises, the first promise which has a value is returned.

### promise.waitAll(...)

Waits until all provided promises have values.

### promise.sleep(ms)

Suspends current fiber for `ms` miliseconds. Does not block the event loop, thus other fibers may execute.
