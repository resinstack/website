---
title: "Assemble, Don't Install"
date: 2022-01-31T21:10:18-07:00
---

One of the common questions that comes up when discussing the
ResinStack is how it differs from more widely known solutions like
HashiCorp Packer.  Lets take a quick look into some of the qualities
of each.

## HashiCorp Packer

Packer is a build automation tool for assembling machine images from
either existing images that require specialization, or from OS
installation media to create entirely new base images.  The way packer
does this is far beyond the scope of this article, but generally
speaking packer performs its magic by scripting the lifecycle of a
virtual machine in some provider, either locally or in a remote
environment.  It then runs one or more provisioners that interact with
the machine to run scripts, configuration management tools, or other
policy application mechanisms.  This allows a great degree of
flexibility because it enables any task that could be done by hand to
also be done to the machine under construction.

All this flexibility comes at a price though, its difficult to apply
machine policy to and generally is only introspect-able once artifacts
have been generated.  This unfortunately results in a very opaque
build and which may result in an unusable artifact.  Packer also
generates environment specific artifacts, which while useful means
that it is not always obvious if a defect being investigated has to do
with the image specifically or the particular environment it has
manifested in.

Perhaps the biggest requirement of packer is also its greatest
strength, it depends on the operating system it is installing to be
"self-hosting."  In this case, self-hosting refers to an operating
system that is capable of installing itself and further installing
software using included tooling.  Packer has no visibility into the
operating system it is installing, and likewise the operating system
being installed has no visibility into packer.

## LinuxKit

LinuxKit is a small suite of utilities all referred to collectively as
linuxkit which are used for building packages, assembling OS images,
and post processing those OS images into artifacts suitable for
specific virtualization platforms.

The way linuxkit does what it does is nuanced and complicated, and
you're highly encouraged to read [the linuxkit architecture
document](https://github.com/linuxkit/linuxkit/blob/master/docs/architecture.md)
if you are interested in the gritty details.  In short though,
linuxkit starts from nothing and constructs an n-way filesystem union
from pre-prepared filesystem images to produce the root filesystem of
a virtual machine.  This image is then exported as a kernel and packed
filesystem in initrd format.  These artifacts can be further
post-processed for a variety of providers, or used as is, but the
important part is that at no point does this process require actually
booting the final artifact.

This has some great advantages, but does come at a cost.  First off,
it means that as long as your initial input artifacts are well
versioned, you can reproduce the same OS images later.  This is
important for audit-ability as well as debugging problems that may only
be discovered significantly farther down the road.  Second, this
approach masks broken images that are only broken at boot time by not
booting them.  Its important to use a robust workflow that includes
tests when using a machine image from an assembly based paradigm.

So what are the advantages of this approach?  Well, besides the image
reproduce-ability feature, not booting the image means it doesn't
technically have to even be bootable on the current architecture!  For
example, if your cloud provider provides multiple machine
architectures such as amd64 and arm64, you likely don't want to have
to duplicate your entire build infrastructure for the second
architecture.  At the very least this would require some clever use of
`qemu` to do machine instruction translation across architectures.
This is a fascinating but low performance approach to building OS
images for other platforms.  Instead it would be much nicer to take
artifacts that have been pre-prepared for other architectures and
assemble them together into a complete build pipeline.  A quick aside:
its assumed here that any organization that is seriously able to use
multiple incompatible CPU architectures in production has the
capabilities to build their production applications in multiple
languages due to the use of modern languages that either are platform
agnostic (python, nodejs, ruby) or capable of low/zero effort cross
compiling (golang, rust, haskell).

Assembling images rather than booting them also means that the
assembly process is much more declarative.  There's no need to run
commands, so the need to lex shell scripts to figure out what's being
changed isn't necessary.  Instead its possible to introspect spec
files directly to determine if policy infractions are present.  Since
the ResinStack uses terraform to assemble machine images, its possible
to use existing tooling such as tfsec or checkov to apply custom rules
for validation.  Such introspection and rules validation is typically
intensely OS specific since it has to be applied after the final
artifact is constructed, but when assembly is used instead of
installation it becomes possible to evaluate the individual components
in isolation.

---

So when should you use each?  When its appropriate!  If you require
the services of a general purpose operating system such as the need to
run custom installation scripts on a per-machine basis then a tool
like Packer is a great fit.  Packer can also be a great fit for when
you need to use less modular operating systems such as macOS or
Windows where minimal base images are otherwise unavailable.  If
though you are able to confine your on-machine needs to workflows
where the base image can be kept small, relatively simplistic, and
stateless, then the linuxkit paradigm offers compelling advantages.

This post has hopefully provided some illumination on why the
ResinStack uses linuxkit, and how it is different from traditional
tools like packer.  Feel free to reach out via any of the modes of
contact on the [contacts](/contact/) page if you have any questions or
feedback.

