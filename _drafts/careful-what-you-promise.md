---
layout: post 
title: Careful what you promise
categories: tech
tags: javascript async
---

## Background

I was working recently on a JavaScript front-end project that uses ES6 `async` / `await` syntax.
We created a client to some back-end services hosted on AWS Lambda and AWS API
Gateway.

The client is successful at fetching temporary auth tokens and credentials from specified
endpoints, using the credentials to call other services, and automatically refreshing tokens or
credentials when they are about to expire.

## New feature

We wanted another feature, to retry any calls to services, including those that return
the auth tokens and credentials. The aim is to retry a limited number of times, with increasing
time to wait between requests.

Our code calls services using the [Axios](https://github.com/axios/axios) library, which returns
a promise. A simple example method is:

```javascript
const getData = async (url, config) => {
  const { data } = await axios.get(url, config);
return data;
}
```  

(An Axios promise rejection is thrown as an exception that is caught elsewhere.)

## Solution

A simple solution is to wrap the Axios call in a retry loop, like this:

```javascript
/* eslint-disable no-await-in-loop */
const delays = [50, 100, 200, 400, 800, 1600, 3200, 6400];
const waitWithRetries = async (promise) => {
  for (const delay of delays) {
    try {
      // Returning the promise directly does not wait for it in this case.
      const result = await promise;
      return result;
    } catch (error) {
      // The axios docs state that the error object will have a response
      // attribute if status was outside the valid range [200, 300) by
      // default. But there is no response attribute so we retry all
      // failures.
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error(`AwsServiceClient: timeout after ${delays.reduce((a, b) => a + b)} ms`);
};
/* eslint-enable no-await-in-loop */
```

The `getData` function is upated 
