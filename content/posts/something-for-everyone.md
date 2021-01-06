---
title: "Something for Everyone"
date: 2021-01-05T23:31:07-08:00
---

In adapting the ResinStack to run on hardware (blog post coming soon)
some new knowledge was gained, and this is that a one size fits all
solution is incredibly difficult to actually make, and the trade-offs
come fast and numerous.

To this end, the ResinStack needs to change tack slightly from being a
golden master image to being a set of clear templates and tools for
building the system image that contains a site's common configuration
baked in.  What is common configuration?  Things like CA certificates,
whether or not a site wants to use an external Vault for PKI, and
similar customizations that would be difficult to support by other
means.  This leads to the logical choice then that the resulting image
is not the important part of the project, but rather the clear
instructions of how to build the image.

Of course its important to not introduce additional tooling, which is
why the ResinStack project is pleased to release a terraform provider
for linuxkit, so that the builds can be driven by terraform.  The
provider and more documentation can be found on [the terraform
registry](https://registry.terraform.io/providers/resinstack/linuxkit/latest).
