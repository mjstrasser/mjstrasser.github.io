
--
layout: post
title: Kotlin extensions
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

The functions `also` and `apply` return the receiver. For example, this is a natural way to get a
Jackson object mapper and configure it in one expression:

```kotlin
val mapper = jacksonObjectMapper().apply {
    configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
    configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, false)
}
```

If the receiver is not used inside the lambda, `also` is natural:

```kotlin
return serviceResponse.also {
    log.info("Relevant log message.")
}
```


