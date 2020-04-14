---
layout: post
title: Tracing Spring asynchronous code with New Relic – a better way
categories: tech
tags: spring-boot tracing new-relic java
---

In my [earlier post about tracing Spring asynchronous code with New Relic]({% post_url
2020-03-16-tracing-spring-async-with-new-relic %}) I showed a simple solution using a
subclass of `ApplicationEvent` to carry a New Relic token. It has some disadvantages:

1. Code that uses it must explicitly declare New Relic tracing using the `@Trace`
   annotation, must create subclasses of `TracedEvent` and must call the
   `TracedEvent#linkToken` method on the event object.
   
2. Each token can only be expired once, even if an event is listened to by multiple
   listeners.

# A better way

This method uses an implementation of `java.util.concurrent.Executor` that wraps a
delegate instance.

1. The `NewRelicTraceExecutor#execute` method is called in the parent thread. It constructs
   a `TracedRunnable` that wraps the `Runnable` instance it is given.

2. The `TracedRunnable#run` method is called in the child thread. It calls `Token#linkAndExpire` method
   before calling `run` on its delegate `Runnable`.
   
All the New Relic-specific code is in this one class, which can be wired into a Spring Boot
application to be used with `ApplicationEventMulticaster`. Each event listener has its own
`Runnable` instance with its own New Relic token.

```java
package com.example.tracing;

import com.newrelic.api.agent.NewRelic;
import com.newrelic.api.agent.Token;
import com.newrelic.api.agent.Trace;

import java.util.concurrent.Executor;

public class NewRelicTraceExecutor implements Executor {

    private final Executor delegate;

    public NewRelicTraceExecutor(Executor delegate) {
        this.delegate = delegate;
    }

    @Override
    public void execute(Runnable command) {
        Token token = NewRelic.getAgent().getTransaction().getToken();
        delegate.execute(new TracedRunnable(command, token));
    }

    static class TracedRunnable implements Runnable {

        private final Runnable delegate;
        private final Token token;

        TracedRunnable(Runnable delegate, Token token) {
            this.delegate = delegate;
            this.token = token;
        }

        @Trace(async = true)
        @Override
        public void run() {
            token.linkAndExpire();
            delegate.run();
        }
    }
}
```

As before, there is a dependency on the New Relic API. In Gradle:

```groovy
    implementation 'com.newrelic.agent.java:newrelic-api:5.11.0'
```
