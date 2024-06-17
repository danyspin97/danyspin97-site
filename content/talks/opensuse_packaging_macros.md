---
title: 'openSUSE packaging: is macro the way to go?'
date: "2024-04-23T09:00:00"
categories: ["workshop"]
tags: ["opensuse", "packaging"]
video: https://youtu.be/XHL7PkGZWGo?feature=shared
---
class: center, middle
name: start

## openSUSE packaging
### Is macro the way to go?

.green[.footer[![:i](fab fa-suse) SUSE Labs]]
---
background-image: url(/img/opensuse_packaging_macros/chamaleon.jpg)
---
class: left, middle

### %whoami

### Danilo Spinella</br>_Software Engineer in Packaging_

---
class: left, middle

## %contents

Packaging C programs

Packaging Python programs

Caveats

Improvements

---
class: left, middle

## RPM openSUSE packaging

---
class: left, middle
background-image: url(/img/opensuse_packaging_macros/rpmspec.png)

<!--https://build.opensuse.org/projects/home:dspinella/packages/mdp/files/mdp.spec?expand=1 -->

---
class: center, middle

## What is RPM spec?

It is a collection of metadata, commands and macros

---
class: center, middle

## ![:i](fa fa-box-open) Packaging

It is a collection of metadata, sources, commands and patches

---
class: left, middle

## Distributions

![:resize 48](/img/opensuse_packaging_macros/arch.svg) Arch Linux

![:resize 48](/img/opensuse_packaging_macros/exherbo.svg) Exherbo

![:resize 48](/img/opensuse_packaging_macros/void.svg) Void Linux

---
background-image: url(/img/opensuse_packaging_macros/pkgbuild.png)
<!-- https://gitlab.archlinux.org/archlinux/packaging/packages/mdp/-/blob/main/PKGBUILD?ref_type=heads -->

---
class: right, middle
background-image: url(/img/opensuse_packaging_macros/exherbo.png)
<!-- https://gitlab.exherbo.org/DanySpin97/danyspin97-exheres/-/blob/master/packages/app-misc/mdp/mdp-1.0.15.exheres-0 -->

---
class: left, middle

mdp-1.0.15.exheres-0

---
background-image: url(/img/opensuse_packaging_macros/voidtemplate.png)
<!-- https://github.com/void-linux/void-packages/blob/949796da5ac909d0c9a3866d004cf3aca93246b4/srcpkgs/mdp/template -->

---
background-image: url(/img/opensuse_packaging_macros/rpm-makefile.png)

---
background-image: url(/img/opensuse_packaging_macros/exherbo-makefile.png)
<!-- https://www.exherbolinux.org/docs/eapi/exheres-for-smarties.html#src_compile -->

---
background-image: url(/img/opensuse_packaging_macros/void-makefile.png)
<!-- https://github.com/void-linux/void-packages/blob/master/common/build-style/gnu-makefile.sh  -->

---
class: center, middle

# Thoughts ![:i](fa fa-comment-dots) 

---
class: left, middle

## Packaging python programs ![:i](fab fa-python)

---
<!-- https://gitlab.archlinux.org/archlinux/packaging/packages/khard/-/blob/main/PKGBUILD?ref_type=heads -->
background-image: url(/img/opensuse_packaging_macros/arch-python-1.png)

---
background-image: url(/img/opensuse_packaging_macros/arch-python-2.png)

---
<!-- https://build.opensuse.org/projects/devel:languages:python/packages/python-khard/files/python-khard.spec?expand=1 -->
background-image: url(/img/opensuse_packaging_macros/opensuse-python-1.png)

---
background-image: url(/img/opensuse_packaging_macros/opensuse-python-2.png)

---
<!-- https://github.com/danyspin97/exheres/blob/master/packages/app-misc/khard/khard-0.13.0.exheres-0 -->
background-image: url(/img/opensuse_packaging_macros/exherbo-python.png)

---
<!-- https://github.com/void-linux/void-packages/blob/master/srcpkgs/khard/template -->
background-image: url(/img/opensuse_packaging_macros/void-python.png)

---
class: left, middle

## Caveats

---
background-image: url(/img/opensuse_packaging_macros/opensuse-update.png)

---
<!-- https://gitlab.exherbo.org/exherbo/arbor/-/blob/58ef9a71b7bd414394bf6a7ba65b4a953733a6d7/exlibs/python.exlib -->
background-image: url(/img/opensuse_packaging_macros/exherbo-update.png)

---
<!-- https://github.com/danyspin97/exheres/blob/a70af92f8e92f702a02aa9da7154289f6ed7e736/packages/app-admin/yadm/yadm.exlib#L32 -->
background-image: url(/img/opensuse_packaging_macros/exherbo-exlib-loading.png)

---
background-image: url(/img/opensuse_packaging_macros/bash-1.png)

---
background-image: url(/img/opensuse_packaging_macros/bash-2.png)

---
background-image: url(/img/opensuse_packaging_macros/bash-3.png)

---
class: left, middle

## Other caveats

Exponential number of macros, exlibs or build-styles

Packaging 90% of applications is the same across all distributions

---
class: left, middle

## Improvements

---
class: left, middle
<!-- https://www.oilshell.org/blog/2021/01/why-a-new-shell.html#how-is-oil-different-than-bash-or-zsh -->
background-image: url(/img/opensuse_packaging_macros/oilshell.png)

Oilshell

---
class: left, middle
<!-- https://github.com/danyspin97/wpaperd/blob/main/install.yml -->
background-image: url(/img/opensuse_packaging_macros/rinstall.png)

rinstall

---
class: left, middle
background-image: url(/img/opensuse_packaging_macros/nix.png)

NixOS ![:resize 32](/img/opensuse_packaging_macros/nix-logo.png)

<!-- https://github.com/NixOS/nixpkgs/blob/8451d7f4ca93ae1740afa5d2d1cb451c6b9dbd5b/pkgs/applications/misc/khard/default.nix#L2 -->

---
class: left, middle

Distribution agnostic packaging format

---
class: left, middle

## Thank you for your attention

# Questions ![:i](fa fa-question)

---
template: start

