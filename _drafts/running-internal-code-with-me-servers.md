---
layout: post
title: Running internal Code With Me servers
categories: tech
tags: jetbrains intellij
---

JetBrains is developing [Code With Me](https://www.jetbrains.com/code-with-me/) that performs a very similar
function to [Visual Studio Code Live Share](https://code.visualstudio.com/learn/collaboration/live-share):

> Code With Me is a new collaborative development and pair programming service. 
> It enables you to share the currently opened project in your IDE with others, 
> and work on it together in real time.

It has been in EAP for a few months and seems pretty useful.

Code With Me uses central servers to link users and exchange information. There are two kinds of servers:

- a **lobby server** that connects parties who want to code together; and
- one or more **relay servers** that connect hosts and guests in case P2P connections don’t work or are forbidden.

By default, Code With Me uses servers provided by JetBrains, but it is possible to set up your own.

# Running servers in a closed environment

I have been working with a large company that uses JetBrains IDEs and has switched to almost-exclusively
remote work. Like many companies, there was some concern about the potential for proprietary code and other sensitive
information to leak outside if external servers are used. 

The company has a capable internal platform for deploying services so setting it up was easy. By default, services
deployed on it are only visible to users on the internal network and VPN.

I also wanted to test the idea that relay servers may not be necessary, at least to begin with, for internal
company users. Relay servers are not needed if user computers have direct network routes to each other. 
The VPN hosts are all on the private `172.16.0.0/12` address range so it seemed possible.   



The [Architecture overview and on-premises server quick setup](https://jetbrains.com/help/cwm/code-with-me-quick-setup.html)
and the [Code With Me administration guide](https://jetbrains.com/help/cwm/code-with-me-administration-guide.html)
explain how that can be done.



The lobby server is required but relay servers may not be required if P2P connections work.


## Lobby server in XXX

I have deployed the lobby server to XXX to experiment with using only P2P connections, which is the simpler
configuration. If P2P connections work over the XXX VPN there is probably no need for relay servers.


```Dockerfile
FROM debian:buster-slim

ADD dist/lobby /home/lobby-server

RUN apt-get update && apt-get install -y unzip net-tools procps && apt-get clean

WORKDIR /home/lobby-server

ENV JAVA_HOME /home/lobby-server/jbr
ENV SERVER_PORT 8080
ENV ENABLED_FEATURES p2p_quic,direct_tcp

ENTRYPOINT ["bin/lobby-server"]

EXPOSE 8080
```
