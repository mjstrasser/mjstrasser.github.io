---
layout: post
title: How many concurrent users?
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

This model helped the client move from a “so many customers at once” mindset
to a realistic idea of how many concurrent users there would actually be.

Here is an example model. A company has xxx users in offices across 8 regions:

| Region          | Number |
|-----------------|-------:|
| North America   | 12,000 |
| Europe          |  9,000 |
| United Kingdom  |  6,500 |
| Africa          |  4,500 |
| Latin America   |  2,000 |
| China           |  5,500 |
| Southeast Asia  | 11,000 |
| India           |  8,500 |


## Notes

* The relationship between number of concurrent users and system request rate (e.g.
  requests per second) can be complicated. The former is intuitive but can be\
  difficult to measure
  while the latter is easier.
* Concurrent users can be estimated from systems that manage user sessions
  (i.e. sign in and sign out) and (I assume) by web analytics services. 
* How do you count concurrent users? If a user leaves a browser tab open for days
  without actively using a site, do they count as an active user? Are they included
  in a concurrent user count? 
* Request rate is often calculated as a metric by tools like AWS CloudWatch and
  New Relic.

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

