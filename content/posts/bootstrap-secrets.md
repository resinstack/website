---
title: "Bootstrap Secrets"
date: 2020-05-27T22:16:48-07:00
---

Bootstrapping is a hard problem.  With the Hashistack this problem is
harder, as there are dependency loops and no clear agreement on where
to break them.

For our purposes, there are two main bootstrapping loops:

  * Vault depends on Consul for storage.  While it is possible to use
    a built-in Raft database with Vault, this is still very much a
    tech preview, and is not yet suitable for production use.  This
    also does not work well with the opinion of the project that
    Consul is the point of cluster state and topological knowledge.

  * Nomad needs Consul to be available to perform automatic
    clustering.  Because Consul is providing the backing storage to
    Vault, and for best-practices reasons, ACLs must be enabled.  This
    means that now Nomad needs a token to perform automatic
    clustering.  Ideally it would get this token from Vault, but this
    early in initialization that isn't practical, so we need some way
    to get the token securely.


Really what we need here is an external secrets store.  This secrets
store could be a Vault server that is external to the cluster, or
something provided by a cloud service.  In the ResinStack reference
implementation, this is fulfilled by the AWS Secrets Manager.  This
service allows us to securely escrow just enough data to break the
bootstrapping loops.  We still don't want to need to go on-box to use
these secrets once they become available though, so its necessary to
poll for the values.

Such is the job of the `emissary` task on ResinStack machines.  This
task starts up as a service after the network has been configured and
polls a configured secrets store to fill in values in templates.  Once
a template has been completely rendered `emissary` can execute a
command, such as reloading a service.

The `emissary` task allows all-in-one control planes to be
incrementally bootstrapped as secrets become available.  The emissary
task is simple and fairly single purpose.  Its use cases are not
limited to the ResinStack though, and if you have a use case
extension, patches are welcome to support new secret backends.
