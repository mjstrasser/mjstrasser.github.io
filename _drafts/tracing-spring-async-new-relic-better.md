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
   annotation and must also call the `linkToken` method on the event object.
   
2. Each token can only be expired once, even if an event is listened to by multiple
   listeners.

# A better way

This method uses an implementation of `java.util.concurrent.Executor` that wraps a
delegate instance. Comments in the following code describe how it works.

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
            try {
                token.link();
                delegate.run();
            } finally {
                token.expire();
            }
        }
    }
}
```

