---
layout: post
title: Octopus Deploy server in AWS and polling tentacles
categories: tech
tags: aws octopus-deploy
---

I am using [Octopus Deploy](https://octopus.com/) on a current project where the deployment targets
are all in tightly-controlled environments. We have decided to use [polling
tentacles](https://octopus.com/docs/administration/security/octopus-tentacle-communication) to remove
the need to get ingress firewall rules manually created for every deployment target.

# Polling tentacles

We are deploying Octopus Deploy server into [AWS](https://aws.amazon.com/) and need to configure
it securely to accept these connections from polling tentacles to:

* the Octopus web portal when installing the tentacle
* the Octopus server for tentacle instructions, usually on port 10943

The first connection is HTTP or HTTPS and can be secured simply in AWS with any load balancer that
presents a certificate and offloads TLS, forwarding requests to Octopus server on port 80.

The second connection is HTTPS but must be secured from end to end. On installation, both server and tentacle
generate a self-signed certificate, which they use to secure
all communication between them. This means that an AWS load balancer must not offload the certificate
but must pass the traffic unchanged to the Octopus Deploy server.

# AWS Load Balancers

The newer versions of AWS [Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/index.html)
come in two types: [Application Load
Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) and
[Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html).

Application Load Balancers can route traffic based on host, header, path etc. and are very powerful.
But they can only accept HTTP and HTTPS connections and always offload TLS in the latter case.

Network Load Balancers do not support complex routing rules but allow TCP passthrough of requests.
This solution meets our needs:

* A listener on port 443 offloads the TLS certificate on requests to the web portal, which are
  forwarded over HTTP.

* A listener on port 10943 passes requests through unchanged to port 10943 to the same server. 
  
## Security Group differences

* ALB: SG is applied to the LB
* NLB: SB is applied to the destination services 
