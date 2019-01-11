---
title: "First Steps With 66-exherbo"
date: 2019-01-09T23:14:50+01:00
draft: true
---

Developed by obarun[^4], 66 is a new collection of tools built around `s6`[^2] and `s6-rc`[^3],
their scope is well-defined: making easier to use the above init and service
manager and improve administration of the system.

In this post we will follow the installation of 66 and necessary files to boot
your system. We will then learn about trees and how to manage them.

***Note***: _every bash command written here is assumed to be run as root, unless
explicitly noted._

## Installation

### Uninstalling systemd

**Note**: _Skip this part if you have already uninstalled `systemd`_.

66 replace the default init and service manager `systemd`, so we need to get rid
of it before installing. First uninstall it by giving the following command:

```bash
cave uninstall systemd -u '*/*'
```

`-u` stands for `--uninstalls-may-break` which let paludis uninstalls
packages which removal could break the system; in this is case
it is harmless if we update the system before a reboot.

Now add the following options in `/etc/paludis/options.conf`:

```
*/* providers: eudev -systemd
*/* -systemd
```

Do not resolve `world` yet, since we will add other options right after this
step.

### Add 66 options

Unmask `66-exherbo` (init policies for 66) by adding the following to
`/etc/paludis/package_unmask.conf`:

```
app-admin/66-exherbo[~scm]
```

The following option will both add 66-exherbo and install service file for
installed packages:

```
*/* providers: 66
*/* parts: 66
```

Update the world:

```bash
cave resolve world -c -x
```

and select `66-exherbo` as init:

```
eclectic init set 66-exherbo
```

## First steps

The most important concept for 66 is the tree: a tree is a collection of
services that can be handled together. You can add how many services you want to
a specific tree, then you can manage them in a bulk.

The `boot` tree is already created at installation time and contains all the
service to make the system working. It must not be enabled and not be changed
unless you know what're doing and are fully aware that the system could also not
work anymore.

Since `boot` cannot be changed, we need a new tree where to add new services.
I'll call this tree `extra`, outside of this introduction, choose meaningful
names for the set of services you create.

```bash
66-tree -n extra
```

This new system tree is empty, you can check it by running:

```bash
66-info -T extra
```

Let's add some service, say for example `connman` (which I'm assuming is
installed on the system):

```bash
66-enable -t extra connman
```

Default command line arguments for `connmand` are put in
`/etc/66/env/connman/CMDARGS`; in the same folder you will find files containing
settings for the service. It is automatically populated when the service gets
enabled.

Start up the tree:

```bash
66-all -t extra up
```

If you want to avoid adding the name of the tree to every command issued, mark
the tree as current:

```bash
66-tree -c extra
```

The `extra` tree is ready and running. But what if we want to start this tree
at boot time? `66-exherbo` start all the enabled tree after `boot` tree finished.
All that is left is enabling the tree:

```bash
66-tree -E extra
```

You can also target a specific service of a tree at time. Assuming `extra` as
the current service, the three following commands will respectively start, stop
and reload `connman`:

```bash
66-start connman
66-stop connman
66-start -r connman
```

## User services

## Other important commands

* Remove a tree:
```bash
66-tree -R tree_name
```

