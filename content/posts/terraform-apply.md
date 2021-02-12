---
title: "Terraform Apply"
date: 2021-02-12T11:22:47-08:00
---

As touched on in the last post, building images is a critical part of
immutable infrastructure.  Trying to ship a single image that could
satisfy all possible users isn't a practical goal, so the decision was
made to change out the makefile at the heart of the ResinStack for a
more composeable approach.

When removing the makefile, decisions had to be made about what to
replace it with.  There were a few options that could have won out,
lets take a look at what those were:

  * Custom Program - LinuxKit's preferred format for configuration is
    YAML, and it would have been relatively simple to build a program
    that did similar to the makefile in assembling multiple fragments
    of yaml to achieve the final configuration passed to `linuxkit
    build`.  This approach has some shortcomings though.  It doesn't
    lend itself well to immediate understanding, which is a major goal
    of the project.  It also doesn't satisfy the goals of
    maintainability as an upstream, as it makes the mistake of
    reaching for a new tool as the first answer.

  * Really Good Documentation - Since ultimately the tooling is just
    merging yaml dictionaries in complex ways, its possible to just
    provide really good documentation of how to do this by hand.  This
    is similar to how you might consider Linux From Scratch to be a
    "distribution" as by some definitions a distribution is just
    instructions for how to assemble various components into a working
    system.  This solution is pretty annoying though because it
    doesn't make it easy to use the software, and requires fairly deep
    knowledge of how LinuxKit actually works internally.  It also
    requires some manual work any time you want to build a new image.

  * jsonnet - When considering any problem its important to
    rules-lawyer it to know all possible solutions.  One possible
    solution is that most yaml parsers can parse json to some degree.
    It would be possible to use a tool like jsonnet to interpolate and
    merge in various fragments to assemble a final configuration to
    stream to the build process.  This "solution" has a lot of
    problems, one being that it requires anyone who wants to extend it
    to learn an entirely new templating language that really isn't
    particularly human readable.  JSON is after all a machine format,
    not a human one.  Another problem with this approach is that it
    isn't designed behavior of most yaml parsers, so its relying on
    more or less luck to actually work.

In the end, Terraform was selected as the system of choice for building
images.  Terraform has several advantages over anything listed above,
lets look at just a few:

  * Familiarity in the target user group.  Most folks working with the
    HashiStack already have heard of terraform, and its one of the
    cleaner ways to maintain the intertwined ACLs when running Nomad,
    Consul, and Vault in concert.

  * Clear distribution mechanism.  Re-usable terraform modules have a
    clear distribution mechanism.  Modules are pushed to the terraform
    registry which the terraform CLI can natively request data from.
    Overall this is a very straightforward solution to both produce
    and consume.

  * HCL2 is a configuration language that supports all the levels of
    extensibility that are required to build and extend the
    ResinStack.  Custom files and services are easy to add, and its
    relatively trivial to extend the provided images.

So what does this look like in practice?  Lets take a look at an
example where we build a system image that can be PXE booted that
contains Nomad, Consul, and Vault:

```hcl
module "aio_server" {
  source  = "resinstack/resinstack/linuxkit"
  version = "0.1.0"

  system_version_metadata   = "2cf1db0f0d2c9916b4894318bd76f1c97d8c8f7b"
  system_metadata_providers = ["metaldata"]

  enable_console = true
  enable_sshd    = true
  enable_ntpd    = true

  consul_server = true
  consul_acl    = "deny"

  nomad_server  = true
  nomad_client  = true
  enable_docker = true
  nomad_acl     = true

  vault_server      = true
  vault_tls_disable = true

  output_to = "${path.root}/out"
  base_name = "aio"
  build_pxe = true
}
```

The above snippet is all it takes.  You can put this in a file, call
`terraform init` and `terraform apply` and assuming that you have
docker and linuxkit available on your host, you'll get a set of files
that you can load on a TFTP server and feed to a PXE boot script.

This is all good and well for a standard image, but what if we want to
add custom services?  Prometheus is a popular monitoring tool, lets
add the node_exporter:

```hcl
data "linuxkit_image" "node_exporter" {
  name = "node_exporter"
  image = "linuxkit/node_exporter:v0.8"
}

module "aio_server" {
  source  = "resinstack/resinstack/linuxkit"
  version = "0.1.0"

  system_version_metadata   = "2cf1db0f0d2c9916b4894318bd76f1c97d8c8f7b"
  system_metadata_providers = ["metaldata"]

  enable_console = true
  enable_sshd    = true
  enable_ntpd    = true

  consul_server = true
  consul_acl    = "deny"

  nomad_server  = true
  nomad_client  = true
  enable_docker = true
  nomad_acl     = true

  vault_server      = true
  vault_tls_disable = true

  custom_services = [data.linuxkit_image.node_exporter.id]

  output_to = "${path.root}/out"
  base_name = "aio"
  build_pxe = true
```

It really is that simple!  Anything that you can contain in a docker
image can be added to the build, but keep in mind that these images
are immutable, and if you want to change their contents you'll need to
rebuild them and somehow get your machines to use the new images.

The terraform module is available today from the registry and is
suitable for production use, but as with all upgrades you are strongly
encouraged to review the changes and to ensure your environment will
not be adversely affected.
