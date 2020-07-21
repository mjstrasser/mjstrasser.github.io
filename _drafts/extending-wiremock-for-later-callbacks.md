---
layout: post
title: Extending WireMock for later callbacks
categories: tech
tags: testing test-double kotlin
---

Recently I was working with [WireMock](http://wiremock.org/) to provide test doubles for a system
we were testing.

Our system under test calls APIs that return after a short delay with simple information.
WireMock is ideal to mock this kind of API. It is easy to configure with delays using its
`/__admin/mappings` endpoint.

# Asynchronous APIs

This system also calls asynchronous APIs that return immediately with an acknowledgement and
call back after a delay that is much longer, often as much as 10 minutes. These APIs conform
to a contract, where the initial request includes the callback URL.

