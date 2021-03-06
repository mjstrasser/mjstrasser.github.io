---
layout: post
title: Tracing Spring asynchronous code with New Relic
categories: tech
tags: spring-boot tracing new-relic java
---

I have been working with Spring Boot microservices in an environment that is monitored
using [New Relic](https://newrelic.com). Applications instrumented by New Relic are
deployed with agents that send status and other information
to a central server for monitoring and analysis. 

New Relic’s [Distributed Tracing](https://docs.newrelic.com/docs/understand-dependencies/distributed-tracing/get-started/introduction-distributed-tracing)
enables complex request flows to be traced through multiple services
instrumented with its agents. This is a powerful tool for quickly finding
interesting or anomalous traces so they can be examined. We instrumented the Spring Boot
services with New Relic and were able to follow synchronous calls made to downstream 
services. 

# The problem

But it didn’t trace all calls to other services. We executed some code asynchronously using Spring’s
[custom application events](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#context-functionality-events).
Events are published by an `ApplicationEventMulticaster` configured with a task executor, and subscribed to by [asynchronous
listenters](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#context-functionality-events-async).
We found that New Relic trace context was not being transferred with the events to the 
listeners in different threads. 

When the asynchronous listener code called other services, those services were not recognised by New Relic
as participating in the same distributed trace.

# A simple solution

Our solution was to extend the Spring `ApplicationEvent` class to carry with it a
New Relic trace token, and for the listener code to link that token to its New Relic
context.

## Prerequisite

Include the New Relic agent in the project’s runtime dependencies. In Gradle:

```groovy
    implementation 'com.newrelic.agent.java:newrelic-api:5.10.0'
```

## The `TracedEvent` class

```java
package com.example.events;

import com.newrelic.api.agent.NewRelic;
import com.newrelic.api.agent.Token;
import org.springframework.context.ApplicationEvent;

public class TracedEvent extends ApplicationEvent {
    
    private Token traceToken;
    
    TracedEvent(Object eventObject) {
        super(eventObject);
        traceToken = NewRelic.getAgent().getTransaction().getToken();
    }
 
    public void linkToken() {
        traceToken.linkAndExpire();
    }
}
```

There is no need for null checking on New Relic classes because `NewRelic.getAgent()` always
returns a usable object. When the code executes without an actual
agent connected, it returns an instance of `NoOpAgent` that returns a safe instance
of `Transaction` that itself returns a safe, do-nothing instance of `Token`.

## Listener code

Important parts of the code:

```java
public class SomeEvent extends TracedEvent {
    // etc.
}
```

```java
import com.newrelic.api.agent.Trace;

@Service
public class ExampleListener {

    @Trace(async = true) // Ensure New Relic traces this method’s thread
    public void onEvent(SomeEvent event) {
        event.linkToken(); // Do this first

        // Act on the event
    }
}
```

# Future improvements

This simple solution was adequate for our immediate purposes but is not complete. With
`ApplicationEventMulticaster` an event may be listened to by multiple listeners but the token
will be expired by the first listener that uses it. In our case
each event had only one listener.

It is valid to retrieve multiple tokens from a single New Relic transaction and use each
one independently. We could fetch a token for each
listener or to fetch a token for each thread used by the event multitasker’s task executor.

It is better to use Spring configuration to automatically fetch tokens and use them in new contexts.
[Spring Cloud Sleuth](https://cloud.spring.io/spring-cloud-sleuth) uses this technique to ensure
tracing information is propagated to new threads. 
