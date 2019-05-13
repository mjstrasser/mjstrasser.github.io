---
layout: post
title: Octopus Deploy server in AWS and polling tentacles
categories: tech
tags: aws octopus-deploy
---

I am using [Octopus Deploy](https://octopus.com/) on a current project to deploy to a number of
targets in tightly-controlled, on-premises environments. We are using [polling
tentacles](https://octopus.com/docs/administration/security/octopus-tentacle-communication) so
we don’t need to get ingress firewall rules manually created for every deployment target.

# Tentacle-to-server communication

Octopus Deploy server is deployed into [AWS](https://aws.amazon.com/) and needs to be configured
to securely accept connections from polling tentacles:

* the Octopus web portal on its assigned port
* the Octopus server for tentacle instructions, usually on port 10943

The first connection is HTTP or HTTPS and can be secured simply in AWS with any load balancer that
presents a certificate and offloads TLS, forwarding HTTP requests to the server.

The second connection is HTTPS but must be secured from end to end. On installation, both server and tentacle
generate a self-signed certificate, which they use to secure
all communication with each other. This means the Octopus Deploy server cannot be deployed behind a
device that offloads the TLS certificate.

# AWS Load Balancers

The current generation of AWS [Elastic Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/index.html)
come in two types: [Application Load
Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) and
[Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html).

Application Load Balancers can route traffic based on host, header, path etc. and are very flexible.
But they can only accept HTTP and HTTPS connections and always offload TLS in the latter case.

Network Load Balancers do not support complex routing rules but can offload certificates for some
requests and allow TCP passthrough of others.
This solution meets our needs:

* A TLS listener on port 443 offloads the certificate on requests to the web portal, which are
  forwarded over HTTP.

* A TCP listener on port 10943 passes requests through unchanged to port 10943 to the same server. 
  
## Security Group differences

AWS application and network load balancers work differently with security groups.
Application load balancers have security groups attached to them and apply ingress rules.
In contrast, network load balancers do not have security groups attached; here the 
security rules of target instances apply, using their listening ports.

In our configuration the security group for the server specifies port 10943
for the traffic that passes through the load balancer, and port 80 for the web portal 
traffic.
 