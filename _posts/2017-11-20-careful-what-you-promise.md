---
layout: post 
title: Careful what you promise
categories: tech
tags: javascript async
---

## Background

I was working recently on a JavaScript front-end project using ES6 `async` / `await` syntax.
We created a client to back-end services hosted on AWS Lambda via AWS API
Gateway.

The client is successful at fetching temporary auth tokens and credentials from specified
endpoints, using the credentials to call signed services, and automatically refreshing tokens or
credentials when they are about to expire.

Our code calls services using the [Axios](https://github.com/axios/axios) library, which returns
promises. A simple example method is:

```javascript
const getData = async (url, config) => {
  const { data } = await axios.get(url, config);
  return data;
}
```  

## A refactoring mistake

I was refactoring the function’s code to add AWS signing headers to the `config` argument before passing it to
Axios. (That task was previously done elsewhere.)
Given a function `addAwsHeaders` that adds the headers, I wrote something like this:

```javascript
const getData = async (url, config) => {
  const { data } = await axios.get(url, addAwsHeaders(url, config));
  return data;
}
```  

But my tests failed with unresolved promises. This is because the `addAwsHeaders` function is
asynchronous (the auth tokens or credentials may be stale and new ones requested) and returns
a promise that my code was not resolving.

A correct versions is:

```javascript
const getData = async (url, config) => {
  const awsHeaders = await addAwsHeaders(url, config);
  const { data } = await axios.get(url, awsHeaders);
  return data;
}
```

It is important to pass the resolved value from the promise, not the promise itself, to the
function.
