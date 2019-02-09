---
layout: post
title: Kotlin scope functions
categories: tech
tags: kotlin
---

I have been working with [Kotlin](https://kotlinlang.org/) recently and really enjoying it 
compared to Java.

One useful feature that I like is the ability to extend objects (classes?). 
These built-in scope functions `let`, `run`, `also`, `apply` and `with` are really useful
and are described well in
[Coping with Kotlin scope functions](https://kotlinexpertise.com/coping-with-kotlins-scope-functions/).

It can be difficult to remember which one to use. I like to think first of whether the function
returns the receiving object or the value of the lambda.

# Return the receiver

The functions `also` and `apply` return the receiver. For example, here is a natural way to get a
Jackson object mapper and configure it in one expression:

```kotlin
val mapper = jacksonObjectMapper().apply {
    configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
    configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, false)
}
```

Within the lambda, `this` is set to the object returned by `jacksonObjectMapper()`.

Similarly, `also` returns the receiver and passes it to the lambda as an argument instead
of as `self`. It feels natural for side-effects that do not change the receiver, like logging:

```kotlin
return serviceResponse.also {
    log.info("Returning service response: $it")
}

// or alternatively:

return serviceResponse.also { resp ->
    log.info("Returning service response: $resp")
}
```

# Return the value of the lambda

The functions `let`, `run` and `with` return the lambda. They can be used as idiomatic
`null` checks, for example:

```kotlin
val result = value?.let { transform(it) } ?: defaultValue
```

This has the meaning, _If `value` is not null, transform it; else return a default
value._

`run` is similar, except that it applies directly to the receiver (avaiable as `this` in
the lambda).

There is a `with` function like that in Visual Basic, except that it returns the value of
the lambda (which can be ignored).

```kotlin
with(something) {
    setValue(newValue)
}
```
