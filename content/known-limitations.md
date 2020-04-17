---
title: "Known Limitations"
date: 2020-04-16T22:03:39-07:00
---

ResinStack is a new and innovative project.  As such there are certain
limitations in the system right now that may affect your environment.
This page documents these limitations.

## AIO Bootstrap Loops

Right now there are two critical bootstrap loops of an All-In-One
(AIO) control plane.  One loop is formed by using Consul for Vault
storage, which means Vault can't be initialized until after the consul
ACL system is initialized.  This will be resolved by the use of Vault
internal storage, which requires at minimum
[`hashicorp/vault#6409`](https://github.com/hashicorp/vault/issues/6409)
to be resolved.  Additionally, Vault's raft subsystem needs to support
[`go-discover`](https://github.com/hashicorp/go-discover) for truly
automatic clustering.

The second loop is formed via Nomad requiring a Vault token to be able
to issue tokens to tasks running within the cluster.  This loop is
resolved by
[`hashicorp/nomad#7285`](https://github.com/hashicorp/nomad/issues/7258)
or an alternate solution with the same functionality.

## Full TLS Everywhere

Similar to the AIO bootstrapping loops, running with TLS everywhere is
not currently supported due to issues with needing to provision
certificates prior to a certificate provisioning system being
available.  This can be worked around by generating certificates
external to the cluster and including them in the configuration
archive, the security of which is left as an exercise to the reader.

## Current Docker Versions

The docker version is locked to the version available from LinuxKit,
which typically lags by some amount behind the most recent version.
It is possible that ResinStack will maintain a disjoint set of repos
from those of LinuxKit to resolve this shortcoming.

## Lack of Documentation

This is an easy one to fix.  If you are interested in helping write
documentation, [contact us](/contact/).
