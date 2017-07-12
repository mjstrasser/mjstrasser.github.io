---
layout: post
title: Packaging a Spring Boot application into an RPM with Gradle
categories: tech
---

Some learnings from using the
[Gradle Linux Packaging Plugin](https://github.com/nebula-plugins/gradle-ospackage-plugin).

Recently I was working with a client where we were developing new
[Spring Boot](https://projects.spring.io/spring-boot/)
[microservices](https://en.wikipedia.org/wiki/Microservices)
to be deployed to [Amazon Web Services](https://aws.amazon.com/)
VMs. The services were being built using [Gradle](https://gradle.org/)
and needed to be packaged as [RPMs](https://en.wikipedia.org/wiki/Rpm_(software))
for deployment by [AWS CloudFormation](https://aws.amazon.com/cloudformation/) scripts.

I started with a Gradle configuration copied from somebody else’s project and
cut it down as I figured out what worked an what didn’t, and what was actually
needed.

