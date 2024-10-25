---
title: 'Packaging: Analizziamo il DNA delle distro'
date: "2024-10-26T10:45:00"
categories: ["workshop"]
tags: ["opensuse", "packaging"]
---
class: center, middle
name: start

# Packaging
### Analizziamo il DNA delle distro

---
background-image: url(https://www.stickycomics.com/wp-content/uploads/update_for_your_computer.jpg)
---
class: left, middle

### %whoami

### Danilo Spinella</br>_Linux Research Engineer_ @ SUSE ![:i](fab fa-suse)

---
class: left, middle

## Contenuti

Distribuzioni

Pacchetti

Formati dei pacchetti

---
class: left, middle

## Cos'e' una distribuzione ![:i](fa fa-question)

---
class: left, middle

## Distribuzioni (1/2)

Gestore dei pacchetti

Insieme di pacchetti distribuiti

Impostazioni e customizzazioni

---

## Distribuzioni (2/2)

![:resize 215](https://www.vectorlogo.zone/logos/suse/suse-ar21.svg)
![:resize 250](https://www.vectorlogo.zone/logos/redhat/redhat-ar21.svg)
![:resize 250](https://www.vectorlogo.zone/logos/ubuntu/ubuntu-ar21.svg)
![:resize 270](https://www.vectorlogo.zone/logos/archlinux/archlinux-ar21.svg)
![:resize 225](https://www.vectorlogo.zone/logos/getfedora/getfedora-ar21.svg)
![:resize 250](https://www.vectorlogo.zone/logos/debian/debian-ar21.svg)
![:resize 250](https://www.vectorlogo.zone/logos/alpinelinux/alpinelinux-ar21.svg)
![:resize 250](https://www.vectorlogo.zone/logos/nixos/nixos-ar21.svg)
![:resize 280](https://www.vectorlogo.zone/logos/elementaryio/elementaryio-ar21.svg)

---
class: left, middle

## Gestore dei pacchetti

Installa e rimuove pacchetti

Risolve le dipendenze tra pacchetti

Recupera i pacchetti da una repository remota

---
class: left, middle

## Pacchetto

Un archivio compresso contenente:

- Tutti i file necessari per una applicazione, strumento o libreria
- Metadata che descrive il pacchetto e gli script per installarlo, rimuoverlo e aggiornarlo

---
class: left, middle

## Formato dei pacchetti

RPM

PKGBUILD

ebuild

debian

...e altri 100

---
class: left, middle

## Distribuzioni considerate

![:resize 48](/img/opensuse_packaging_macros/arch.svg) Arch Linux

![:resize 60](https://en.opensuse.org/images/a/a3/OpenSUSE-Logo.svg) openSUSE Tumbleweed

![:resize 48](/img/opensuse_packaging_macros/exherbo.svg) Exherbo

![:resize 48](/img/opensuse_packaging_macros/void.svg) Void Linux

---
background-image: url(/img/packaging_analizziamo_il_dna_delle_distro/mdp_pkgbuild.png)

---

background-image: url(/img/opensuse_packaging_macros/rpmspec.png)

---
background-image: url(/img/opensuse_packaging_macros/exherbo.png)

---
class: left, middle

`mdp-1.0.15.exheres-0`

---

background-image: url(/img/opensuse_packaging_macros/exherbo-makefile.png)

---
background-image: url(/img/opensuse_packaging_macros/voidtemplate.png)

---
background-image: url(/img/opensuse_packaging_macros/void-makefile.png)

---
class: center, middle

## Riflessioni ![:i](fa fa-comment-dots)


---
class: middle, left

## Pacchetti Python ![:i](fab fa-python)

---
background-image: url(/img/opensuse_packaging_macros/arch-python-1.png)

---
background-image: url(/img/opensuse_packaging_macros/arch-python-2.png)

---
background-image: url(/img/opensuse_packaging_macros/opensuse-python-1.png)

---
background-image: url(/img/opensuse_packaging_macros/opensuse-python-2.png)

---
background-image: url(/img/opensuse_packaging_macros/exherbo-python.png)

---
background-image: url(/img/opensuse_packaging_macros/void-python.png)

---
class: left, middle

## Problematiche

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

## Altre Problematiche

Numero esponenziale di macro, exlib e build-styles

Creare pacchetti per il 90% delle applicazioni e' uguale tra tutte le distro

---
class: left, middle

## Miglioramenti

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

## Grazie per l'attenzione

# Domande ![:i](fa fa-question)

---
template: start

