---
layout: post
title: Running private Code With Me servers
categories: tech
tags: remote-work aws jetbrains
---

JetBrains is developing [Code With Me](https://www.jetbrains.com/code-with-me/) that performs a very
similar function to
[Visual Studio Code Live Share](https://code.visualstudio.com/learn/collaboration/live-share):

> Code With Me is a new collaborative development and pair programming service.
> It enables you to share the currently opened project in your IDE with others,
> and work on it together in real time.

It has been in Early Access Program for a few months and works pretty well.

Code With Me uses central servers to connect users with each other. Once connected they 
can communicate directly with each other.

# Running servers in a closed environment

By default, Code With Me uses public servers provided by JetBrains, but you can set up your own.

I have been working with a large company that uses JetBrains IDEs extensively and has almost
exclusively switched to remote work. They could really benefit from using Code With Me but are
understandably concerned to protect their code and sensitive information and might not be
comfortable using external servers.

I wanted to see how easy it would be to run private Code With Me servers on the company’s managed
cloud infrastructure. The company has a platform for deploying containerised workloads on
AWS that made it easy to set up. By default, services deployed on it are only visible to users on
the internal network and VPN.

## Can we use only a lobby server?

Code With Me uses two kinds of servers:

- a **lobby server** that connects parties who want to code together; and
- one or more **relay servers** that connect users in case direct P2P connections don’t
  work or are forbidden.

I also wanted to test the idea that relay servers are not needed for internal company users.
Developers who work together are all on internal company networks or the corporate VPN, which are
both in the private `172.16.0.0/12` address range, so this idea seemed possible.

I modified the instructions in the [Code With Me 
administration guide](https://jetbrains.com/help/cwm/code-with-me-administration-guide.html):

- I downloaded the latest version of the lobby server from the download link on that page. (The
  link takes you to an obligatory name and email form before revealing links to lobby and relay 
  servers.)
- I set up a simplified Dockerfile for the lobby server, reduced from the one on that page:

```dockerfile
FROM debian:buster-slim

ARG DISTRIBUTION_VERSION=1296
ADD lobby-server-linux-x64.${DISTRIBUTION_VERSION}.tar.gz /home/lobby-server

RUN apt-get update && apt-get install -y unzip net-tools procps && apt-get clean

WORKDIR /home/lobby-server

ENV JAVA_HOME /home/lobby-server/jbr
ENV SERVER_PORT 8080
ENV ENABLED_FEATURES p2p_quic,direct_tcp

ENTRYPOINT ["bin/lobby-server"]

EXPOSE 8080
```

Compared with the example in the [administration
guide](https://jetbrains.com/help/cwm/code-with-me-administration-guide.html):

- There was no need for a `config.json` file because we were not setting up relay servers.
- The platform can provision Redis instances for services, so `REDIS_HOST` and `REDIS_PORT` were 
  set in platform configuration. 
- `BASE_URL` was set per environment in platform configuration.
- The platform provisions load balancers with certificates so there is no need for NGINX and 
  certificate configuration.
- Removed `ws_relay` from the list of `ENABLED_FEATURES`.
  
## Did it work?

Yes! 

We tried using `direct_tcp` without `p2p_quic` but it stopped working: the latter was also required
for P2P communication to work in this environment.
