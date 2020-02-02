---
layout: post
title: Modelling user load
categories: tech
tags: testing modelling
---

We often ask the question, “How many concurrent users can my system support?” when
assessing the scalability of a system. 

We often ask questions about numbers of concurrent users that a system can support,
for example:

* “How many concurrent users do we expect?”
* “How many concurrent users can we support?”

# Modelling the number of concurrent users

## Assumptions

* Relationships between regions and time zones (i.e. distributions of people across zones).
* Everyone will use the product on the first day.
* Usage is evenly distributed throughout that day. This can be varied to experiment with
  different models.
* Length of user sessions.

## Relationship between users and API calls

User journey models

* How many steps in a journey?
* How much time between steps?
* How many API calls in each step?  
