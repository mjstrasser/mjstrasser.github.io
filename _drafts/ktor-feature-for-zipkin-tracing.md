---
layout: post
title: Ktor feature to handle Zipkin IDs in HTTP headers
categories: tech
tags: kotlin ktor
---

[Ktor](https://ktor.io/) is a [Kotlin](https://kotlinlang.org/) framework for building asynchronous
servers and clients. It is being actively developed to work with as many Kotlin environments as possible.
I am learning Kotlin and Ktor because they are both interesting [and
topical](https://www.thoughtworks.com/radar#kotlin-klimbing).

# Features

Much functionality of Ktor is implemented using *Features* that are installed [into
servers](https://ktor.io/servers/features.html) and [into
clients](https://ktor.io/clients/http-client/features.html). They are used to insert functionality into
the request pipeline.

An example built-in server feature is [Call Logging](https://ktor.io/servers/features/call-logging.html),
which can be simply installed to log all requests like so:

```kotlin
    install(CallLogging) {
        level = Level.INFO
        filter { call -> call.request.path().startsWith("/api") }
    }
```

# Zipkin

This feature does not [instrument](https://zipkin.io/pages/instrumenting.html) Ktor. It simply generates
and propagates tracing headers.

 
