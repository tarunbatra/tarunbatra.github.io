---
title: Why you should use named JavaScript functions
date: 2017-02-27 00:59:36
tags:
  - javascript
  - styleguide
category:
  - javascript
photos: data/images/why-you-should-use-named-javaScript-functions/cover.jpeg
---

JavaScript functions can be categorized into *named* or *anonymous* on the basis of the value of their __name__ property. A named function can be declared as follows:
```js
function add(a, b) {
  return a + b;
}

console.log(add.name) // -> "add"
```
<br>
All other ways, including much touted ES2015's [Arrow functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions) produce anonymous functions. One of these ways is:
```js
var add = function (a, b) {
  return a + b;
}

console.log(add.name) // -> ""
```
__NOTE__: If you tried the above code in Chrome Dev Tools, you would've noticed that the __name__ of the function is "add" even in the case of anonymous function. That's because modern-day interpreters are smart enough to recognize the name from the expression.

Now, what's special about __named functions__?

## Make sense with error stacks

Error and call stacks are a great help in debugging code. But the use of anonymous functions, reduces the usefulness of the call stacks. We all have seen the infamous __(anonymous)__ littered over the stack trace.

Of course, one can always look at the line number of the error, but with the ever increasing pre-processing on the code (read Babel, WebPack, Browserify), this is not a reliable method.

### Error stack using anonymous function
<p data-height="252" data-theme-id="dark" data-slug-hash="OpVLwZ" data-default-tab="js" data-user="tbking" data-embed-version="2" data-pen-title="Anon Func" class="codepen">See the Pen <a href="https://codepen.io/tbking/pen/OpVLwZ/">Anon Func</a> by Tarun Batra (<a href="http://codepen.io/tbking">@tbking</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

### Error stack using named function
<p data-height="252" data-theme-id="dark" data-slug-hash="mWJdJK" data-default-tab="js" data-user="tbking" data-embed-version="2" data-pen-title="Named Func" class="codepen">See the Pen <a href="https://codepen.io/tbking/pen/mWJdJK/">Named Func</a> by Tarun Batra (<a href="http://codepen.io/tbking">@tbking</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script><script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

JavaScript ecosystem is [already confusing](https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f) as is and the developer can use as much help as she can get to make sense of the obscure errors.

## Use methods before declaration

JavaScript while interpreting code looks for statements starting with __function__ keyword (i.e. named function expressions) and move them on the top, before running the code. This is called [Hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting).

So practically, a named function is available even before it's declared, making the following code valid.

```js
add(1, 2); // => 3

function add (a, b) {
  return a + b;
}
```
But the following code won't work.
```js
add(1, 2); // => Error

var add = function (a, b) {
  return a + b;
}
```

## Better readability

It's fairly difficult to understand a callback function's purpose in first glance. That's what makes [callback hell](http://callbackhell.com/) the problem it is.
Look at the following jQuery code.
```js
$('form').submit(function () {
    // ...
});
```
The above code provides no information on what's going to happen to the submitted form. Adding names to callbacks improves readability and acts as implicit documentation, as the following code does.
```js
$('form').submit(function hitAPI () {
    // ...
});
```

## Conclusion

In my opinion there's no reason why one should not use named functions whenever possible. Most of the JavaScript style guides out there don't give due importance to them. [ESLint](http://eslint.org/) users can use [func-name](http://eslint.org/docs/rules/func-names) rule in their projects to require the use of named functions.

<br>

**❤️ code**
️
