# Async Await

- The newest way to write asynchronous code in JavaScript.
- It is non blocking (just like promises and callbacks).
- Async/Await was created to simplify the process of working with and writing chained promises.
- Async functions return a Promise. If the function throws an error, the Promise will be rejected. If the function returns a value, the Promise will be resolved.

## Syntax

Writing an async function is quite simple. You just need to add the `async`keyword prior to `function`:



## Await

Async functions can make use of the `await` expression. This will pause the `async` function and wait for the Promise to resolve prior to moving on.

we can use it with try catch  block but lets not do that 

 Instead of using` try{}` `catch(e) {}` in each controller, we wrap the function in
  `catchErrors()`, catch any errors they throw, and pass it along to our express middle ware with `next()`

```javascript
exports.catchErrors = (fn) => {
  return function(req, res, next) {
    return fn(req, res, next).catch(next);
  };
};




```

  if there is an error it will catch it and run `next()`

in app.js we have all the error handlers setup. all you have to do is wrap your controller in `catchError()`and use async function and wait your statements.