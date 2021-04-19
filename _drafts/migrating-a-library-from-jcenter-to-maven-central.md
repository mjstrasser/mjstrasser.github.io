---
layout: post
title: Migrating a library from JCenter to Maven Central
categories: tech
tags: gradle githum maven
---

# The problem

When I received notification that [JFrog would shut down the JCenter / Bintray repositories in 
May this year](https://jfrog.com/blog/into-the-sunset-bintray-jcenter-gocenter-and-chartcenter/)
I knew I might have to move [the small Kotlin library I created a couple of years 
ago](https://github.com/mjstrasser/ktor-features-zipkin). The library is an experiment and 
is not production ready (for example, it only conveys tracing IDs
but does not send them to a tracing server) but has a few followers. With only a few weeks to go,
one of the followers asked me if I was going to migrate the library to Maven Central (and offering
to help). So I realised I should get going and fix it.

## Change the build system

Meanwhile, the existing build process that used Travis CI also need to be updated because Travis CI
needs users migrate from https://travis-ci.org to https://travis-ci.com. So I decided to change the
build system to [GitHub actions](https://github.com/features/actions) so everything is in one place.

# The tasks

## Access to Maven

- Application to Sonatype via JIRA
- Proving ownership of michaelstrasser.com

## Gradle

- JavaDoc
- Signing
- Maven POM
- Publishing

## GitHub Actions

This is the simplest part of the process because it is just a sequence of calls to Gradle tasks.

# What did I learn?


