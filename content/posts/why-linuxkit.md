---
title: "Why LinuxKit"
date: 2020-01-02T21:33:36-07:00
---

The ResinStack is built on top of immutable machine images that are
produced with [LinuxKit](https://github.com/linuxkit/linuxkit).
LinuxKit is a system for building immutable images that run
containers.

If you're curious how this differs from things like CoreOS or Flatcar
Linux, the answer is simple: LinuxKit is a mechanism for building
immutable images.  Flatcar Linux has a concept of users, the ability
to interact with it remotely and the ability to change it after it
comes online.  By default, LinuxKit packs your entire system image
into a read-only CPIO archive with no mechanism for external
interaction.  Flatcar is designed to be installed to a disk, whereas
LinuxKit images are designed to boot from a clean source every time.
With the appropriate configuration, it is possible to boot an entire
datacenter from the network using LinuxKit images, though this may be
more or less difficult depending on your particular environment and
network architecture.

LinuxKit has another main distinction between it and other dedicated
container operating systems, and that is minimalism.  Whereas other
options have an entire SystemD init process running in them, LinuxKit
goes for a very small amount of memory safe Go to bring the system up.
The base system images are all based on Alpine, and this lends itself
well to creating a very small boot image.  An all in one image
containing Nomad, Consul, Vault, CoreDNS, Docker, SSH, a getty
process, and a handful of startup tasks is a mere 316M of space.
Being small also comes with another advantage, and that's security.

Often you hear about hardening servers after they've been installed.
If you take a step back to think about what this implies, you realize
that the concept is that your base image is simply insufficient for
use in a secure environment.  Typically this is a trade-off between
convenience and security.  Most environments are built on top of
general purpose Linux distributions, and most general purpose
distributions are meant to be convenient for their users to do things
with.  This convenience comes at a price, and sometimes that price is
security.  The hardening process often consists of turning off
components, adding additional security measures, or trying to remove
base functionality that's not required to serve the function for which
a machine was commissioned.

Wouldn't it be much simpler to skip the hardening process and just
start out with an image that was secure by default?  This is the idea
behind the use of LinuxKit.  By starting at nothing and building up,
we can add only the minimum number of components to the image needed
to accomplish the goal of running Nomad.  If only critical
dependencies have been added, and they've been configured in a secure
manner, then there's nothing left to harden.  ResinStack doesn't need
to have a package manager, so we can get rid of that.  We don't need
to support a lot of complex things such as network accounts, so we
don't need NSS or PAM and can swap out glibc for the much smaller and
better tested musl C library.  We don't need to log into these
machines, so we don't need getty or sshd in a production image.
