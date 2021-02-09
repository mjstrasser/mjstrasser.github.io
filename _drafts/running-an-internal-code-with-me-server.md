---
layout: post
title: Running an internal Code With Me server
categories: tech
tags: jetbrains 
---

## Lobby and relay servers

Code With Me uses central servers to link users and exchange information. There are two kinds of servers:

- a lobby server that connects parties who want to code together; and
- one or more relay servers that connect hosts and guests in case P2P connections don’t work or are forbidden.

The lobby server is required but relay servers may not be required if P2P connections work.

By default it uses servers provided by JetBrains, but it is possible to use different ones.
The [Architecture overview and on-premises server quick setup](https://jetbrains.com/help/cwm/code-with-me-quick-setup.html)
and the [Code With Me administration guide](https://jetbrains.com/help/cwm/code-with-me-administration-guide.html)
explain how that can be done.

## Lobby server in XXX

I have deployed the lobby server to XXX to experiment with using only P2P connections, which is the simpler
configuration. If P2P connections work over the XXX VPN there is probably no need for relay servers.
