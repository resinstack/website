---
title: "An Immutable Future"
date: 2019-12-28T21:10:18-07:00
---

Immutable Infrastructure has become one of the modern buzzword terms
of the systems worlds to describe a method of operating software
systems that once built are unchangeable.  The mechanism for change is
to build a new system and then replace the existing one, effectively
producing an externally visible change.

The ResinStack is a project that aims to enable immutable
infrastructure at the lowest levels for the Hashicorp stack based on
Nomad, Consul, and Vault.  This will be accomplished using LinuxKit,
the same technology that powers Docker for Mac.

The name comes from the idea that the infrastructure is effectively
cast in a block of clear resin.  You can see it, you can interact with
it in limited ways, but you cannot change it non-destructively.  In
this way, the system is quite secure, quite easy to reason about, and
most importantly built on established principles of success from the
world of container orchestration.
