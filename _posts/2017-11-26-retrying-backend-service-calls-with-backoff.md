---
layout: post
title: Retrying back-end service calls with back-off
categories: tech
tags: javascript async promise generator
---

Recently I worked on a single-page ES6 application that calls many back-end
services using the [axios](https://github.com/axios/axios) promise-based
library.

## Back off!

We wanted to implement the *back-off* pattern to retry service calls when they
suffer transient failures. With back-off, the client does not retry immediately.
Instead, it sleeps for a short delay before retrying again. If the same request
continues to fail, the client sleeps for progressively longer delays before each
retry.

There is the [axios-retry](https://github.com/softonic/axios-retry) plugin that
adds retry capability to Axios but it works by counting the number of retries
without providing any back-off delays. 

## A solution

We implemented retry with back-off by looping through an iterable of delay
periods in milliseconds:

```javascript
const retryAxios = async (delays, axiosFunc, ...axiosArgs) => {
  // Extract the iterator from the iterable.
  const iterator = delays[Symbol.iterator]();
  while (true) {
    try {
      // Always call the service at least once.
      return await axiosFunc(...axiosArgs);
    } catch (error) {
      const { done, value } = iterator.next();
      if (!done && isRetriable(error)) {
        await sleep(value);
      } else {
        // The error is not retriable or the iterable is exhausted.
        throw error;
      }
    }
  }
};
```

Notes:

- `delays` is an [ES6
Iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols).
An array is a simple example.

- A `while` loop is used so the service is called at least once. This requires
the iterable to be unpacked manually so the `done` attribute of the object
returned by the iterator’s `next` method can be checked after the service is
called.

- The `sleep` function is the usual promise-based mechanism to wait for a time
period:

```javascript
const sleep = delay => new Promise(resolve => setTimeout(resolve, delay));
```

## Transient failures

We consider network and server errors as transient, thus suitable to be retried:

```javascript
// Network error rules from https://github.com/softonic/axios-retry.
const isNetworkError = error => (
  !error.response
  && error.code
  && Boolean(error.code)
  && error.code !== 'ECONNABORTED'
  && isRetryAllowed(error)
);
const isServerError = error => (
  error.response
  && error.response.status >= 500
  && error.response.status <= 599
);
const isRetriable = error => isNetworkError(error) || isServerError(error);
```

The `isRetryAllowed` function is from the
[is-retry-allowed](https://github.com/floatdrop/is-retry-allowed) module: it
checks the error code against lists of values that can be retried and those that
cannot.

## Back-off implementations

A simple implementation of `delays` is an array, for example:
 
```javascript
const delays = [50, 50, 100, 100, 200, 500, 1000, 1000, 1000, 1000];
```

This example provides ramp-up to repeated 1-second delays.

It is common to use exponentially-increasing delays between retries. This
`ExponentialDelays` class returns an iterable of a specified number of delay
times in milliseconds:

```javascript
class ExponentialDelays {
  constructor(initialDelay, retryCount) {
    this.initialDelay = Math.max(1, initialDelay);
    this.retryCount = Math.max(0, retryCount);
  }
  // Iterator generator
  * iterator() {
    let delay = this.initialDelay;
    for (let retry = 0; retry < this.retryCount; retry += 1) {
      yield delay;
      delay *= 2;
    }
  }
  // Implement iterable protocol.
  [Symbol.iterator]() {
    return this.iterator();
  }
}
```

## Some examples

```javascript
  // GET with simple array of delays.
  const delays = [50, 50, 100, 100, 200, 200, 500, 1000, 1000, 1000];
  const { data } = await retryAxios(delays, axios.get, 'https://api.example.com/people/12345');
```

```javascript
  // POST with exponential set of delays.
  const delays = new ExponentalDelays(10, 6);
  const { data } = await retryAxios(delays, axios.post, 'https://api.example.com/people', {
    firstName: 'Osbert',
    lastName: 'Sitwell',
  });
```

```javascript
  // Using a custom axios instance as a function with 10 equally-spaced delays.
  const customAxios = axios.create({
   timeout: 30000,
   withCredentials: true,
   maxRedirects: 0,
  });
  const delays = new Array(10).fill(100);
  const { data } = await retryAxios(delays, customAxios, {
    method: 'put',
    url: 'http://api.example.com/people/12345',
    data: {
      firstName: 'Sacheverell',
      lastName: 'Sitwell',
    }
  });
```
## A note on ESLint

The code for `retryAxios` shown here violates a number of default
[ESLint](https://eslint.org/) rules (`no-await-in-loop`, `no-constant-condition`
and `consistent-return`). Comments to selectively disable those rules have been
omitted here for clarity, but are present in the complete code [in this
Gist](https://gist.github.com/mjstrasser/69d0d03f0eedb513ae529a0c137800f1).
