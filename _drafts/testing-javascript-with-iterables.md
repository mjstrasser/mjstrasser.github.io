---
layout: post
title: Testing Javascript with iterables
categories: tech
tags: javascript async promise generator
---

I wanted to write tests for the `retryAxios` function described in [my recent post
about retrying back-end service calls]({{ site.baseurl }}{% post_url
2017-11-26-retrying-backend-service-calls-with-backoff %}).

Each call to `retryAxios` needs a mock Axios function that will return a known
sequence of responses, where each response is a promise that either resolves (success)
or rejects (error). 

My solution was to create a function that returns a function for each
request to be mocked:

```javascript
const mockAxios = (...returns) => {
  const iter = returns[Symbol.iterator]();
  return () => {
    const { done, value } = iter.next();
    if (done) {
      throw new Error('mockAxios iterator done');
    }
    return mockRequest(value);
  };
};
```

TBC…

This `mockRequest` function provides a bare-bones implementation:

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
    // HTTP success response or failure, according to status code.
    const response = { status: retVal };
    if (retVal >= 400 && retVal <= 599) {
      throw new HttpError(response);
    }
    return response;
  }
  // Otherwise network error.
  throw new NetError(retVal);
};
```

* On success it returns a `response` object with a status value.
* On failure it throws either an HTTP or network error object.
* It is declared `async` so it wraps the return value in a resolved or
  rejected promise.



