---
layout: post
title: Migrating a library from JCenter to Maven Central – Part 2
categories: tech
tags: gradle github maven
---

# The problem

When I received notification that [JFrog would shut down the JCenter / Bintray repositories in 
May this year](https://jfrog.com/blog/into-the-sunset-bintray-jcenter-gocenter-and-chartcenter/)
I knew I might want to move [the small Kotlin library I created a couple of years 
ago](https://github.com/mjstrasser/ktor-features-zipkin). The library is an experiment and 
is not production ready but has a few followers. 

With only a few weeks to go, one of the followers asked me if I was going to migrate the library to
Maven Central, and offered to help. I realised I should understand how these things work and
migrate it myself.

## Also change the build system

Meanwhile, the existing build process used Travis CI and also need to be updated because Travis CI
needs users migrate from [travis-ci.org](https://travis-ci.org) to
[travis-ci.com](https://travis-ci.com). I had never really understood this build system (which I had
copied from someone else’s project) and felt I should really know what I am doing. I decided to
change the build system to [GitHub actions](https://github.com/features/actions) so everything is in
one place.

# The tasks

[The Central Repository Publishing Guide](https://central.sonatype.org/publish/publish-guide/)
has all the details of what is required. Here is a summary of what I did.

## Apply for access to Maven Central

Maven Central (a.k.a. The Central repository) is the epicentre of the Java package universe and is
where most JVM packages can be found. The primary (only?) mechanism for publishing packages is via
the [Nexus Repository Manager](https://oss.sonatype.org/). To publish my library with
coordinates `com.michaelstrasser:ktor-features-zipkin:<version>` I needed to complete these steps:

1. Create an account at [Sonatype Jira](https://issues.sonatype.org/secure/Signup!default.jspa).

2. Create a [ticket for new open-source project repository
   hosting](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134) with the 
   group ID `com.michaelstrasser` and project and source code URLs. I think the **Central 
   OSSRH** user that responds is automated and it typically take less than half an hour to respond.

3. Prove ownership of `michaelstrasser.com` by adding a `TXT` record to its DNS containing the 
   URL to my Jira issue. This was a simple procedure at CloudFlare, where I manage this domain. 
   After that I added a comment to the issue to re-open it so it progressed.

4. After successfully publishing artefacts to [s01.oss.sonatype.org](https://s01.oss.sonatype.org)
   as instructed, I added another comment to the Jira issue to initiate automatic synchronisation
   to Maven Central. 

You can read [my Jira issue](https://issues.sonatype.org/browse/OSSRH-67094) to see all these 
steps in sequence.

## Gradle

Maven Central requires a higher standard of conformance for projects to be published there, 
including signing of artefacts, complete project POMs and including of JavaDoc.

As a result my Gradle build needed updating.

### Signing



### JavaDoc


### Maven POM


###  Publishing

## GitHub Actions

This is the simplest part of the process because it is just a sequence of calls to Gradle tasks.

It is also simpler than the Travis CI build, which required an external file to contain the 
correct version number that was being published.

### Secrets storage

Private key for signing.

# What did I learn?

## Releases and snapshots

- Releases are immutable; snapshots are mutable 
