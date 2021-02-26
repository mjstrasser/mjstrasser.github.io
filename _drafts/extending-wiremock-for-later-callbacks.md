---
layout: post
title: Extending WireMock for later callbacks
categories: tech
tags: testing test-double kotlin
---

[WireMock](http://wiremock.org/) is a flexible Java test double for HTTP APIs that can be run
in-process for automated testing and also as a standalone application. It
has many features built in but can also be extended. 

# Asynchronous APIs

A little while back I had set up Wiremock in standalone mode for performance testing of a service
that makes a large number of HTTP requests. 

I was working recently with a system also calls asynchronous APIs that return immediately 
with an acknowledgement and
call back after a delay that is much longer, often as much as 10 minutes. These APIs conform
to a contract, where the initial request includes the callback URL.

I was able to create a WireMock extension that 
