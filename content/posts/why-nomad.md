---
title: "Why Nomad"
date: 2019-12-30T21:17:01-07:00
---

One of the questions that seems to always show up when discussing
cluster orchestration systems is "why aren't you using Kubernetes?"
Let's look into that.

## Simplicity

Kubernetes can do many things, and has a very advanced operating
model.  To support this, it has a comparably high level of complexity
and overhead to its operation.  This complexity would mean that in
order to run it on top of an immutable base system you'd need not only
some fairly advanced Linux systems knowledge, but also a full body of
knowledge in Kubernetes.  If there were to be any issue in the stack,
it would be very difficult to diagnose.

In contrast, Nomad is a single binary that is deployed and run
everywhere.  Any extensions to Nomad are implemented as plugins,
either statically installed on a host in the case of task driver
plugins, or increasingly as schedulable tasks that run within Nomad
itself for extending the base capabilities.  Furthermore, Nomad does
not bottle up complexity in its base system.  By moving some
complexity to plugins, some to scheduled tasks, and some out of the
scope of Nomad entirely we're left with a system that is not only of a
simpler design, but one that is more resilient as well due to having
fewer moving parts.

## Surrounding Ecosystem

A cluster scheduler rarely stands on its own.  In almost all cases it
operates in concert with a service discovery layer, a security engine,
and typically as well a distributed DNS component.  Nomad is only one
component of this stack.  For service discovery there is fantastic
integration with Consul, and for secret management integration with
Vault is available.  Each of these 3 services does one thing and does
it well, making it an ideal part of the stack.

## Community

This is first and foremost an open source project.  The choice of
technology then is important as the community comes with it, and the
project must be interesting or useful to the community in some way.
The Nomad community is full of people doing interesting and often
officially unsupported things, and running an immutable stack is an
idea that came out of discussions with a number of Nomad operators.

If your site operates on Nomad and would like to assist with this
project, reach out!

