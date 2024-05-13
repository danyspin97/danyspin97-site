---
title: 'rinstall: a modern make install'
date: "2024-04-23T13:00:00"
categories: ["workshop"]
tags: ["opensuse", "packaging"]
---
class: center, middle
name: start

## rinstall
### A modern "make install"

.green[.footer[![:i](fab fa-suse) SUSE Labs]]
---
background-image: url(/img/rinstall/legacy.png)

---
class: left, middle

```yaml
whoami:
    name: Danilo Spinella
    role: Software Engineer in Packaging
    mail: danilo.spinella@suse.com
    site: danyspin97.org
```

---
class: left, middle

## Contents

Why build a new tool

What does it do?

Integration

---
class: center, middle

## Why ![:i](fa fa-question-circle)

---
background-image: url(/img/rinstall/kanidm-install.png)

---
class: left, middle

## Things to notice ![:i](fa fa-circle-exclamation)

<!--
Prone to error

Repetition across distributions

Maintained by packagers
-->

---
class: left, middle

## Ideas ![:i](fa fa-lightbulb)

<!-- make install / install.sh

meson

cargo install

-->

---
class: left, middle

## make install / install.sh

<!-- Added dependdency for non-C projects

Difficult to get it right

Calling "install" in an automated way

Need to learn GNU Directory Standard
-->

---
class: left, middle

## meson

<!--
New dependency

Need to be integrated in other build systems (cargo) or viceversa
-->

---
class: left, middle

## "cargo install" ![:i](fab fa-rust)

<!--
Only supports binaries and libraries

Does not work with workspaces

Does not follow GNU Directory Standard
-->

---
class: left, middle

## A new standard: install.yml

---
class: center, middle

**Declarative installation file** that hide complexity from the developer

---
class: center, middle

![](https://imgs.xkcd.com/comics/standards.png)

---
background-image: url(/img/rinstall/rinstall-install.png)

---
background-image: url(/img/rinstall/rinstall-command.png)
class: left, middle

---
class: left, middle

## Rust projects

rinstall -> target/release/rinstall

---
background-image: url(/img/rinstall/wpaperd-spec-1.png)

---
background-image: url(/img/rinstall/wpaperd-spec-2.png)

---
background-image: url(/img/rinstall/rinstall-macros.png)

---
background-image: url(/img/rinstall/rinstall-command-2.png)

---
background-image: url(/img/rinstall/wpaperd-pkginfo.png)

---
background-image: url(/img/rinstall/rinstall-tarball.png)

---
background-image: url(/img/rinstall/rinstall-tarball-install.png)

---
background-image: url(/img/rinstall/rinstall-uninstall.png)

---
class: left, middle

## Future improvements ![:i](fa fa-star-half-stroke)

<!--
Polish the user interface

Add file types not already covered

Global testing and adoption on OBS and CI
-->

---
class: left, middle

## Thank you for your attention

# Questions ![:i](fa fa-circle-question)

---
template: start

