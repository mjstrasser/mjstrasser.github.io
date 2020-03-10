---
layout: post
title: Tracing Spring asynchronous code with New Relic
categories: tech
tags: spring-boot tracing new-relic
---

I have been working with Spring Boot microservices in an environment that is monitored
using [New Relic](https://newrelic.com). New Relic uses agents deployed with 
agents that send status and other information
to a central server for monitoring and analysis. 

New Relic’s [Distributed Tracing](https://docs.newrelic.com/docs/understand-dependencies/distributed-tracing/get-started/introduction-distributed-tracing)
enables transactions to be traced through multiple services
instrumented with its agents. This is a very powerful feature that enables
anomalous traces to be found quickly and examined. We instrumented the Spring Boot
services with New Relic and were able to follow synchronous calls made to downstream 
services. 

# The problem

Our services also executed some code asynchronously using [custom application
events](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#context-functionality-events).
Events are published by calling `ApplicationEventMulticaster.multicastEvent()` and subscribed to by [asynchronous
listenters](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#context-functionality-events-async).
We found that New Relic trace state was not being transferred with the events to the 
listeners in new threads executing the code. 

When that code called other services, those services were not recognised by New Relic
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
        traceToken.link();
    }
}
```

There is no need for null checking on New Relic classes because `NewRelic.getAgent()` always
returns a usable object. When the code executes without the
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

    @Trace(async = true) // Signal to New Relic to trace this method
    public void onEvent(SomeEvent event) {
        event.linkToken(); // Do this first

        // Act on the event
    }
}
```

That is all you need to do for New Relic to include code executed by the listener and
called by it in the same thread.
