---
layout: post
title: Simple retry with back-off
categories: tech
tags: javascript async 
---

Recently I worked on a single-page application that makes calls to many
back-end services using the [Axios](https://github.com/axios/axios)
promise-based library.

We wanted to implement the **back-off** pattern to retry service calls 
when they suffered transient failures. There is the [Axios retry
library](https://github.com/softonic/axios-retry) that adds retry capability to Axios
but it works by counting the number of retries without providing any back-off
delays.

Instead, we implemented simple retry with back-off by looping through an iterable of delay
periods in milliseconds:

```javascript
const retryAxios = async (delays, axiosFunc, ...axiosArgs) => {
  for (delay of delays) {
    try {
      return await axiosFunc(...axiosArgs);
    } catch (error) {
      if (isRetriable(error)) {
        await sleep(delay);
      } else {
        throw error;
      }
    }
  }
  throw new Error('retryAxios: timeout failure');
};
```

A simple implementation of `delays` is an array, such as `[50, 50, 100, 100, 200, 500]`, which
provides 6 retries for a total delay of 1 second.

This `ExponentialDelays` class returns an iterable of delay times in milliseconds:

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

The `sleep` function uses a timeout promise:

```javascript
const sleep = delay => new Promise(resolve => setTimeout(resolve, delay));
```

The `isRetriable` function specifies that network retries and server errors can be
retried:

```javascript
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

Here the `isRetryAllowed` function is from the [is-retry-allowed](https://github.com/floatdrop/is-retry-allowed)
module, that checks an error code against a lists of values that can be retried
and those that cannot.
