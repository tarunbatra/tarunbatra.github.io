---
title: Pitfalls of Mocha's built-in Promise support
date: 2017-04-19 02:11:12
tags:
  - javascript
  - tests
category:
  - javascript
photos: data/images/Pitfalls-of-Mocha-s-built-in-Promise-support/cover.png
---

[Mocha][mocha-url] is the leader when it comes to testing frameworks in [Node][node-url], and it's great. I use it in all my projects, and I'm sure many others do too.

Mocha added support for Promises back in version `1.18.0`. Recently while testing asynchronous code written using Promises, I was using the [built-in promise support][mocha-promise] instead of the old-fashioned callbacks oriented [`done` callback][mocha-done] and I found my tests were resulting in **false positives**. So here I'm explaining what happened and how to avoid it.

## Problem
It's easy to see that there are four possible cases when testing Promises:

| Case | Expectation | Actual   | Expected test status |
| :--: | :---------: | :------: | :------------------: |
| 1    | Resolved    | Resolved | Pass                 |
| 2    | Resolved    | Rejected | Fail                 |
| 3    | Rejected    | Rejected | Pass                 |
| 4    | Rejected    | Resolved | Fail                 |

### Case 1
```js
var promise = Promise.resolve();

describe('the promise resolves expectedly', () => {
  it('should PASS the test', () => {
    return promise.then(() => {
      console.log('Promise resolved!') //  Called
    });
  });
});
```
#### Test status: **PASS**
The test expects the promise to resolve and so it does. The snippet above shows that the test passes expectedly.

### Case 2
```js
var promise = Promise.reject(new Error('some-error'));

describe('the promise rejects unexpectedly', () => {
  it('should FAIL the test', () => { // => Error: some-error
    return promise.then(() => {
      console.log('Promise resolved!') // Not called
    });
  });
});
```
#### Test status: **FAIL**
In the case when test expects the promise to resolve, but it doesn't, Mocha gracefully fails the test. In the snippet above, the `then` block was never executed and Mocha detected the error rejected by the code and failed the test.

### Case 3
```js
var promise = Promise.reject(new Error('some-error'));
describe('the promise rejects expectedly', () => {
  it('should PASS the test', () => {
    return promise.catch(() => {
      console.log('Rejected promise caught!');  // Called
    });
  });
});

```
#### Test status: **PASS**
The test expects the promise to be rejected and it is caught by the `catch` block, which results in this test passing as expected.

### Case 4
```js
var promise = Promise.resolve();

describe('the promise resolves unexpectedly', () => {
  it('should FAIL the test', () => {
    return promise.catch(() => {
      console.log('Rejected promise caught!') // Not called
    });
  });
});
```
#### Test status: ~~FAIL~~ **PASS**
In this case, the test expects the promise to be rejected and the catch block to execute but instead, it doesn't. The promise resolves. Mocha doesn't throw an error here!

So while doing [Negative Testing][negative-testing], the built-in promises support doesn't hold up well and can result in **false positives**. This is not a bug and the reason behind this behavior totally makes sense, once given a careful thought, but at the very least, it's not intuitive and causes the tests to pass even when they shouldn't.

## Solution

### Good ol' done callback
The simplest way to avoid this situation is to always handle the rejection scenarios yourself, rather than asking Mocha to do it. Here's how:
```js
var promise = Promise.resolve();

describe('the promise resolves unexpectedly', () => {
  it('should FAIL the test', (done) => {
    promise.catch(() => {
      console.log('Rejected promise caught!') // Not called
      done();                                 // Timeout!
    });
  });
});
```
#### Test status: **FAIL**
The above test will fail due to timeout as Mocha waits for `done` to be called but it doesn't, and times out eventually.

### Chained then block
Another way to avoid this situation is to chain a `then` block to the existing `catch` block and assert that the catch was called.
```js
var expect = require('chai').expect;
var spy = require('sinon').spy;
var exceptionHandler = spy();
var promise = Promise.resolve();
describe('the promise resolves unexpectedly', () => {
  it('should FAIL the test', () => {
    return promise.catch(exceptionHandler)
    .then(() => {
      // Assert that catch block was called
      expect(exceptionHandler.calledOnce).to.be.true;
    });
  });
});
```
#### Test status: **FAIL**
The above test tries to assert that the catch block was called using [Chai][chai-url] and [Sinon][sinon-url], and fails.

If you know other elegant ways to avoid this behavior, please let me know in the comments!

[mocha-url]: https://mochajs.org
[node-url]: https://nodejs.org
[chai-url]: http://chaijs.com/
[sinon-url]: http://sinonjs.org/
[mocha-promise]: https://mochajs.org/#working-with-promises
[mocha-done]: https://mochajs.org/#asynchronous-code
[negative-testing]: https://en.wikipedia.org/wiki/Negative_Testing

<br>
**❤️ code**
