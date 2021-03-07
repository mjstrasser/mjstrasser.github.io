---
layout: post
title: Deploying this blog on IPFS
categories: tech
tags: blog ipfs
---

I have been experimenting with deploying this blog on the [Interplanetary Filesystem,
IPFS](https://ipfs.io) as well as its usual address.

# IPFS

With IPFS, resources are identified by their contents, not their locations. This means
a new ‘version’ of a page or blog is identified differently because its contents
have changed.

For example, two versions of the home page of this blog (`/index.html`) have two identities:

- `QmTUNULbQQ5wbZWxpXe2WNt93DdkiHxcN18Trk1LEGJBrd` for the version published from Git
  hash `2fc9428e291803f7f06994a4de6adcace2f1afee` on 21 February 2021. 
  
- `QmXce8twurwMLdL85hXQQ3vQFJVLMg2fqNFSxmGsz5qp3t` for the version published from Git
  hash `7143480932428a8d7b38ccadf9590caa446e5d12` on 7 March 2021.

## IPFS hash encodings

The common, compact encoding of hashes is [base-58](https://en.wikipedia.org/wiki/base58), like
those shown above. This encoding is not URL-safe so there is a base-32 alternative.

The base-58 encoding `QmXce8twurwMLdL85hXQQ3vQFJVLMg2fqNFSxmGsz5qp3t` can be converted into the
base-32 encoding `bafybeiej2hj64mw6derjjhp6mqzppru4ouwrnsay2leru6qubtsf5ewsum`.

## Finding resources

I don’t know much about …

I am not interested in the cryptocurrency side of IPFS.

# Deploying with Fleek

[Fleek](https://fleek.co) offers a number of services, including deployment of blogs
to IPFS and managing DNS entries, so these addresses direct to the latest published
version:

- [https://floral-glade-5684.on.fleek.co](https://floral-glade-5684.on.fleek.co)

- [ipns://blog.michaelstrasser.com](ipns://blog.michaelstrasser.com) (only in browsers that
  support IPFS)

# Does it work?

Sometimes. 

Propagation can be slow.

Some Fleek issues; support on Slack.
