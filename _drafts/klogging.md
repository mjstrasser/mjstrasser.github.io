---
layout: post
title: Klogging
categories: tech
tags: klogging
---

Last year I started building a new logging library for Kotlin called
[Klogging](https://klogging.io). I believe there is need for a Kotlin-specific logging library that
offers features not currently or easily available with other options.

Klogging is intended to be multiplatform, but is currently only available on JVM.

# Klogging features

## Pure Kotlin

It is built in Kotlin and depends only on standard Kotlin multiplatform libraries. Klogging makes
use of Kotlin features, especially coroutines.

Kotlin coroutines provide two advantages to Klogging: contextual information and asynchronous
handling of log events.

- Coroutines run in scopes and carry context. Klogging can include information held in coroutine
  contexts in log events.
- Coroutines enable simple and effective asynchronous programming. Klogging uses coroutine
  [channels](https://kotlinlang.org/docs/channels.html) to move log events between the coroutines
  that emit, dispatch and send them.

## Log events not strings

It is valuable to log _events_, not just strings of text. The Java logging frameworks
[Apache Log4j](https://logging.apache.org/log4j) and [Logback](https://logback.qos.ch/)
fundamentally log text, with structured information available via add-ons like
[Logstash Logback Encoder](https://github.com/logstash/logstash-logback-encoder) and
[Log4j JSON Template Layout](https://logging.apache.org/log4j/2.x/manual/json-template-layout.html).

In Klogging, structured events are first-class objects. Each event contains at least timestamp and
local context information (logger and thread names). It can also contain a map of items that
represent the situation when the log event happened and the context in which it happened.

When running in coroutines, Klogging can automatically include contextual information in log events.
That information is available for the duration of the coroutine scope where it is set, and in any
nested scopes inside that one.

Additionally, Klogging implements [Message templates](https://messagetemplates.org/) to make it easy
to add items to structured events.

## Be precise

Historically, Java has recorded log events with millisecond precision. Logback has that baked into
the core of the library (the not-yet-final version 1.3 removes that limitation).

More than once I have encountered multiple log events with the same recorded timestamp and not been
able to know the order in which they were produced.

# What are the alternatives?

There is no standard logging component of the Kotlin standard library. Other approaches to logging
from Kotlin, such as [kotlin-logging](https://github.com/MicroUtils/kotlin-logging), wrap SLF4J.

## On the JVM

Many Java applications use [Simple Logging Facade for Java (SLFJ)](https://www.slf4j.org/) for
logging, primarily because frameworks like [Spring](https://spring.io/) and [Ktor](https://ktor.io)
for JVM.

SLF4J provides a standard API for creating loggers and sending log events. The standard back-end for
SL4J is Logback, but others are available, including Log4j.

Klogging has its own [SLF4J binding](https://github.com/klogging/slf4j-klogging) that is a drop-in
replacement in existing

## Multiplatform

The [Kermit multiplatform logging utility](https://github.com/touchlab/Kermit) is pure Kotlin and
targets all platforms. It logs only text strings without structure and does not offer functions with
access to coroutine contexts for contextual logging.
