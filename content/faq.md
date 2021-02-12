---
title: "Frequently Asked Questions"
date: 2020-09-08T01:11:59-07:00
---

## What is the ResinStack?

The ResinStack is a distribution of Linux specifically tailored for
running the Hashistack.  The distribution is independent and not based
on any other distro.  Builds are produced from-scratch using LinuxKit
and are able to leverage Docker Trust signatures for enhanced
integrity verification.

In addition to the core image, the ResinStack team provides base
Terraform modules useful for bootstrapping your Nomad, Consul, and
Vault policies, as well as modules useful for deploying the ResinStack
into AWS.

## Is this production ready?

How good is your operations team?

If your workloads consist primarily of stateless docker containers, it
should be relatively safe to mix ResinStack workers in with your
compute fleet.  Tasks that rely on state or other binaries being
present on the host probably aren't a good fit for ResinStack machines
right now.

## Is it secure?

Like all software, you need to consider what your risk tolerances are.
That being said, the ResinStack images make use of modern software,
namespaced systems, and the principal of least bits.

In the case of the reference Terraform modules, these are prepared in
an additive process adding permissions and capabilities until the
desired functionality works, and then verifying that no unintended
access was added in the process.

## Does the ResinStack team have access to my clusters/data?

... What? (Yes, this has actually been asked.)

The ResinStack is a Linux distribution.  The authors have exactly as
much access to your clusters and data as if you were running Ubuntu.
Really there's even less access since the ResinStack does not report
statistics back, and because you can reasonably build your own copy of
the entire system image.

Of course this is not an excuse to slack off on securing your clusters
and environments.  If you deal with particularly secure data, you
should take steps to ensure that no-unaudited access to the internet
can occur.

If you have security folks that somehow believe that running open
source software grants the authors access to your data, you really
need to question other things those folks have been telling you.

## Doesn't using pre-built policies make it easier for me to get hacked?

The policies that are provided in the reference Nomad, Consul, and
Vault Terraform modules are based on applying the principal of least
bits to getting the HashiStack to run.  You can create equivalent
policies by reading the getting started guides for Nomad, Consul, and
Vault respectively.

Using these policies is of course not required and you can run the
ResinStack without them loaded at all.  The primary cost to doing this
is that you'll need to duplicate a lot of boilerplate policy.

Of course all of this is predicated on the idea that you aren't
exposing your cluster's control API to the internet.  These APIs are
not intended to be publicly visible.  The ResinStack provides basic
building blocks, it does not provide a turn-key solution that will fit
all situations, your operations group is still responsible for setting
up good firewall rules, auditing the security of your environments,
and staying up to date on software patches.

## How do I get a shell on a ResinStack box?

By default neither SSH nor TTY shells are enabled.  If you require
these features you can build images that include them, but you should
carefully weigh this need vs the security trade-offs.  The ResinStack
was designed to require no direct access to the on-disk system, and if
you believe you need this access you may be approaching an XY problem.
Consider reaching out on Gitter and asking for another way to
accomplish your goals.

## Why can't I download a ready to use system image?

The ResinStack is based on the ideas of immutable infrastructure and
minimalism.  Because every organization is different, its not possible
to solve the needs of every organization while remaining minimal.
Instead, the images can be easily built using the tooling provided by
the ResinStack project.

## Is this only for cloud environments?

No!

You can run the ResinStack on any amd64 compatible environment that
you can load the images into.  If your environment supports PXE
booting, you can use a kernel and initrd to boot the OS directly from
the network.

The one clear limitation that you may need to solve comes from an
implicit dependency on a cloud-init datasource.  Your environment must
provide cloud-init compatible metadata on boot for the
auto-configuration of the system images to work.

## I have a question that wasn't answered here!

That's great!  Questions further the project, and you can ask yours
via any means listed on the [contact page](/contact/).
