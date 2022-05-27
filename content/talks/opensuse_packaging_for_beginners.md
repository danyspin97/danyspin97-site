---
title: 'openSUSE Packaging for Beginners'
date: "2022-06-03T13:00:00"
categories: ["workshop"]
tags: ["opensuse", "packaging"]
---
class: center, middle
name: start

# openSUSE Packaging </br> for Beginners

.footer[3rd June</br>open![:i](fab fa-suse) Conference 2022]
---
class: left, middle

# Danilo Spinella

### _Software Engineer in Packaging_

### **SUSE** ![:i](fab fa-suse)

---
class: left, middle

## Prerequisites

Compiling a simple C program

Basic knowledge of Linux distributions

---
class: left, middle

## ![:i](fas fa-list) Contents

Packaging basics

Create our first package

---
class: left, middle

# Packaging basics

---
class: left, middle

A Linux system is made from **thousands of programs** running together

---
class: left, middle

How do we get a working Linux system ![:i](fas fa-question)

---
class: left, middle

Get the software source code

Compile and install it

Repeat

???
Mention LFS

---
class: left, middle

A binary linux distribution provides the **compiled packages**

---
class: left, middle

A package manager allows the user to _install_/_upgrade_/_remove_ the package

---
class: left, middle

## RPM

Fedora ![:i](fab fa-fedora) and openSUSE ![:i](fab fa-suse) package format

---
class: left, middle

## OpenBuildService (OBS)

Platform to develop openSUSE packages

---
class: left, middle

OBS web interface

`osc`, OBS CLI

---
class: left, middle

# ![:i](fas fa-terminal) Practical part

---
class: left, middle

## ![:i](fas fa-info) Info

This is a workshop, so there will be the time to replicate all the things shown here

If you have any question, just raise your hand

If you have any technical problem, please wait for the slotted time

---
class: left, middle

This is an example slotted time

.footer[.green[![:i](fas fa-clock)]]

---
class: left, middle

## Setup (1/2)

openSUSE distribution installed (preferred) or a system with docker

A working internet connection

An openSUSE account (click [here](https://idp-portal.suse.com/univention/self-service/#page=createaccount)
to create)

---
class: left, middle

## Docker ![:i](fab fa-docker)

`$ docker run -it opensuse/tumbleweed`

---
class: left, middle

## Setup (2/2)

``` bash
# zypper install osc
```

Add your user/password in `~/.config/osc/oscrc`

---
class: left, middle

This presentation is available here:

https://danyspin97.org/talks

.footer[.green[![:i](fas fa-clock)]]

---
class: left, middle

# Creating our first package

---
class: left, middle

## `progress`

Linux tool to show progress for cp, mv, dd

https://github.com/Xfennec/progress

---
class: left, top

Create the package on OBS

Click on "Your Home Project", then "Create Package"

--

Check it out locally

```bash
$ osc co home:<username> progress
$ cd home:<username>/progress
```

---
class: left, middle

## Spec file

A file containing the metadata of a package and how to build it

`progress.spec`

---

Name of the package

`Name:           progress`

--

Version that we are currently packaging

`Version:        0.16`

---

The release version, automatically set by OBS

`Release:        0`

--

A small (less than 80 characters) description of the package

`Summary:       Linux tool to show progress for cp, mv, dd`

---
class: left, middle

License of the package, from [SPDX](https://spdx.org/licenses/) list

`License:        GPL-3.0-only`

---
class: left, middle

## Macros

Macros will be expanded at runtime by rpm

`%{name}` -> **progress**

`%{version}` -> **0.16**

You can check them by using

`$ rpm --eval "%make_build"`

---

The homepage

`URL:            https://github.com/Xfennec/%{name}`

--

The sources and files needed to build and install

`Source:         %{name}-%{version}.tar.gz`

---
class: left, middle

Makefile

```make
ifeq ($(UNAME), Linux)
    ifeq (, $(shell which $(PKG_CONFIG) 2> /dev/null))
    $(error "pkg-config command not found")
    endif
    ifeq (, $(shell $(PKG_CONFIG) ncursesw --libs 2> /dev/null))
    $(error "ncurses package not found")
    endif
    override LDFLAGS += $(shell $(PKG_CONFIG) ncursesw --libs)
endif```

---

Build dependencies

`BuildRequires: pkgconfig`

`BuildRequires: pkgconfig(ncurses)`

--

Lenghty description of the package and what it does

```
%description
a Tiny, Dirty C command that looks for coreutils basic commands (cp, mv, dd, tar, gzip/gunzip,
cat, etc.) currently running on your system and displays the percentage of copied data. It can
also show estimated time and throughput, and provides a "top-like" mode (monitoring).
```

---

Start the *preparation* part of the spec file, where sources will be unpacked and patches will be applied

`%prep`

--

Automatically unpack the sources and apply patches

`%autosetup`

---
class: left, middle

Start the build phase

`%build`

Build by calling make

`%make_build`

---
class: left, middle

Makefile

```makefile
PREFIX ?= /usr/local
BINDIR = $(PREFIX)/bin
MANDIR = $(PREFIX)/share/man/man1
```

---
class: left, middle

```make
install : $(OBJ)
	@echo "Installing program to $(DESTDIR)$(BINDIR) ..."
	@mkdir -p $(DESTDIR)$(BINDIR)
	@install -pm0755 $(OBJ) $(DESTDIR)$(BINDIR)/$(TARGET) || \
	echo "Failed. Try "make PREFIX=~ install" ?"
	@echo "Installing manpage to $(DESTDIR)$(MANDIR) ..."
	@mkdir -p $(DESTDIR)$(MANDIR)
	@install -pm0644 $(OBJ).1 $(DESTDIR)$(MANDIR)/ || \
	echo "Failed. Try "make PREFIX=~ install" ?"
```

---
class: left, middle

Start the install phase

`%install`

Install by using `make install`

`%make_install`

---
class: left, middle

List all the files contained by the package

`%files`

`%{_bindir}/%{name}`

`%{_mandir}/man1/%{name}.1%{?ext_man}`

---
class: left, middle

Needed by RPM but not used by openSUSE

`%changelog`

---
class: left, middle

Download the tarball

`$ wget 'https://github.com/Xfennec/progress/archive/refs/tags/v0.16.tar.gz'`

---
class: left, middle

## Build the package

---
class: left, middle

[CLI version](https://openbuildservice.org/help/manuals/obs-user-guide/cha.obs.basicworkflow.html#sec.obs.basicworkflow.setuphome)

---
class: left, middle

Web version

Click on "Your Home Project", then "Repositories", "Add from a Distribution"

Add *openSUSE Tumbleweed*

---
class: left, middle

`$ osc build`

.footer[.green[![:i](fas fa-clock)]]
---
class: left, middle

## Changes file

Contains the history of the package

---

`$ osc vc`

Add the following line:

`Initial packaging of version 0.16`

--

`$ osc commit`

.footer[.green[![:i](fas fa-clock)]]

---
class: center, middle

# Questions ![:i](fas fa-question)

---
class: center, middle

#![:i](fab fa-creative-commons) ![:i](fab fa-creative-commons-by) ![:i](fab fa-creative-commons-sa)

## This work is licensed under

## _Attribution-ShareAlike 4.0 International</br> (CC BY-SA 4.0)_

---
template: start

