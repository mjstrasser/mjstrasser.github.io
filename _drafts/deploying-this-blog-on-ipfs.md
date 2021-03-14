---
layout: post
title: Deploying this blog on IPFS
categories: tech
tags: blog ipfs
---

I have been experimenting with deploying this blog on the [Interplanetary Filesystem
(IPFS)](https://ipfs.io) as well as its usual address.

# IPFS

With IPFS, resources are addressed by their contents, not their locations. This means a new version
of a page or blog is addressed differently because its contents have changed.

For example, two versions of the home page of this blog (`/index.html`) have two [content
identifiers (CIDs)](https://docs.ipfs.io/concepts/content-addressing/):

- `QmTUNULbQQ5wbZWxpXe2WNt93DdkiHxcN18Trk1LEGJBrd` for the version published from Git hash 
  [2fc9428](https://github.com/mjstrasser/mjstrasser.github.io/tree/6b494eaf9cd4908f42908365014d2d6d3b28c24b) on 21 February 2021. 
  
- `QmXce8twurwMLdL85hXQQ3vQFJVLMg2fqNFSxmGsz5qp3t` for the version published from Git hash
  [7143480](https://github.com/mjstrasser/mjstrasser.github.io/tree/7143480932428a8d7b38ccadf9590caa446e5d12) on 7 March 2021.
  
These CIDs are expressed in base58 encoding; there is a range of other
formats and encodings.

## Finding resources

I don’t know much about …

I am not interested in the cryptocurrency side of IPFS.

# Deploying with Fleek

[Fleek](https://fleek.co) offers a number of services, including deployment of blogs
to IPFS and managing DNS entries, so these addresses direct to the latest published
version:

- [https://floral-glade-5684.on.fleek.co](https://floral-glade-5684.on.fleek.co)

- [ipns://blog.michaelstrasser.com](ipns://blog.michaelstrasser.com), an address that
  is only resolved by browsers that support IPFS, either natively or via extensions.

# Does it work?

Sometimes. 

Propagation can be slow.

Some Fleek issues; support on Slack.
