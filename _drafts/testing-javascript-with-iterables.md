---
layout: post
title: Testing Javascript with iterables
categories: tech
tags: javascript async promise generator
---

I wanted to write tests for the `retryAxios` function described in [my recent post
about retrying back-end service calls]({{ site.baseurl }}{% post_url
2017-11-26-retrying-backend-service-calls-with-backoff %}).

Rather than mocking Axios itself with either [moxios](https://github.com/axios/moxios)
or [Axios mock adapter](https://github.com/ctimmerm/axios-mock-adapter), I wanted a
simple mock of an Axios function. The mock needs to be configurable to return
a known sequence of responses, where each response is a promise that
either resolves or rejects. 

My solution was to create a function that returns a closure:

```javascript
const mockAxios = (...returns) => {
  const iter = returns[Symbol.iterator]();
  return () => {
    const { done, value } = iter.next();
    if (done) {
      throw new Error('Testing error: mockAxios iterator done');
    }
    return mockRequest(value);
  };
};
```

* The [rest parameter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)
  `returns` contains an array that specifies how each function call will return.
* `mockAxios` returns a closure that iterates through `returns`, calling `mockRequest` for
  each value in that iterable.
* If the iterable is exhausted the test itself is in error.

`mockRequest` is a bare-bones implementation of the behaviour of Axios functions:

```javascript
class HttpError extends Error {
  constructor(response, ...params) {
    super(...params);
    this.response = response;
  }
}

class NetError extends Error {
  constructor(code, ...params) {
    super(...params);
    this.code = code;
  }
}

const mockRequest = async (retVal) => {
  if (Number.isInteger(retVal)) {
    const response = { status: retVal };
    if (retVal >= 400 && retVal <= 599) {
      throw new HttpError(response);
    }
    return response;
  }
  throw new NetError(retVal);
};
```

* If `retVal` is an HTTP success status code, it returns a `response` object with that code.
* If `retVal` is an HTTP failure status code, it throws a simplified HTTP error object
  with that code.
* Otherwise it returns a simplified network error object with code `retVal`.
* `mockRequest` is `async` so it wraps the return value in a resolved or
  rejected promise.

Here is a simple example of using `mockAxios` in a [Jest](https://facebook.github.io/jest/)
test case, with inline comments for annotation:

```javascript
test('retryAxios should return first successful call if within retry limit', async () => {
  // Calling expect.assertions() is useful when testing asynchronous code.
  expect.assertions(1);
  const data = await retryAxios(
      // The iterable with delays after retriable failures.
      [5, 5],
      // A mock Axios function that returns server failure, then network failure,
      // then success.
      mockAxios(500, 'ECONNRESET', 301)
  );
  expect(data.status).toBe(301);
});
```

The test code has been added to my [Gist for
retryAxios](https://gist.github.com/mjstrasser/69d0d03f0eedb513ae529a0c137800f1).
