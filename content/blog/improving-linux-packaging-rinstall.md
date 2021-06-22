---
title: "Improving Linux Packaging: rinstall"
date: 2021-06-23T15:45:00+02:00
categories: ['development']
tags: ['packaging', 'linux']
---

Let's face it. Packaging software on Linux is an underappreciated job. If they do their job
correctly, almost no one will notice them. But the moment they do a mistake, the users of their
distribution will likely go mad.

They often work with no standard tool for build and install. They have to
dig into other people source code daily and fix errors regarding build and dependencies.
Likewise, they also have to discuss and teach developers about how distributions and standard
works. Knowing Linux hierarchy, where to place files and a lot of unwritten rules to make systems
work is surely out of the scope for most developers. And I think they have all the rights to skip
this knowledge. They develop software, they don't build systems (but there are exceptions too!).

However, developers are (almost) always asked to write the installation target for their software,
may it be a shell script or a complex C codebase. Here, tools often don't help the developers,
as they don't properly support or enforce the various standards.
Therefore, package maintainers have to fill the gap themselves. It usually happens in their own
distribution package specification. How many distributions are there? Tons. And this
work is duplicated countless times.

While developing software written in Rust, I have found a gap in `cargo-install`: it doesn't
consider any data or file outside of executables and libraries. In spite of that, applications
usually have many files to install; man pages, documentation, examples, `.desktop`
files and icons. What do developers do in this case? They either add a `Makefile` or entirely
skip the installation target. This is something that happens not only to Rust projects, but
to many others types.

I have engineered a tool to fill the gap, **[rinstall]**, and this post is about its rationale,
usage and how it could **improve packaging on Linux** (and other *nix systems too!).

[rinstall]: https://git.sr.ht/~danyspin97/rinstall

## rinstall

rinstall is a small (at the time of writing, just ~600 LOC!) helper tool that has two
main objectives:

- Help developers define which files their program shall install without knowing all the various
  standards, hierarchy and the quirks of writing _Makefiles_.
- Remove the duplicated code in the distribution packages definitions that manually use `cp` and
  `install` to install files.

It doesn't try to replace tools such as `meson`, that already wonderfully covers the installation
phase. It doesn't try to replace either `CMake` or `make` when they are already used for
compiling in C/C++ projects. Although they don't do any enforcing, they are already used,
so it doesn't make a lot of sense to add a new dependency on a new tool.

I am sure most of you are ready to throw knives at me the moment I said I want to replace
`Makefiles` in some contexts. I know what you're feeling, but please hold on and continue reading.
There will be an in-depth comparison and explanation. If you still aren't satisfied by then,
I won't stop you anymore!

### Install directories

**rinstall** supports two installation modes out of the box: **system-wide** and **user** install.

The first installs into `/usr/local` by default, making the programs available to all system users.
It follows the _[Directory Variables]_ from the _GNU Coding Standard_. Binaries will be installed
into `/usr/local/bin`, libraries into `/usr/local/lib` and so on. The prefix can be changed by
passing it into the command line, as well as all the other variables.

[Directory Variables]: https://www.gnu.org/prep/standards/html_node/Directory-Variables.html

The second installs into the `HOME` folder of the current user, following the
_[XDG Base Directory Specification]_. Binaries will be installed into `~/.local/bin`,
libraries into `~/.local/lib`, and all the other variables will use the various `XDG_*` variables
with default fallback.

[XDG Base Directory Specification]: https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html

It also reads the configuration from `/etc/rinstall.yml` when running as root and
`.config/rinstall.yml` when running in user mode. If the configuration file exists,
the tool will merge said config with the default one.

In addition, it also supports a `--destdir` argument. Packagers can just call the following
command to perform the installation phase of a program:

```bash
# rinstall --destdir mydir --prefix /usr --no-config
```

### `install.yml`

**rinstall** resolves around the per-project file `install.yml`. This file contains the
list of files to install, divided per category, as well as the name and version of the software.
For example this is the `install.yml` for **rinstall** itself:

```
name: rinstall
version: 0.1
type: rust
exe:
  - src: rinstall
docs:
  - src: LICENSE.md
  - src: README.md
```

Each outmost key (`exe` and `docs` in this case) contains a list of sources, abbreviated `src`,
and optionally the destination, abbreviated `dst`. **rinstall** will install the source into either
the folder or file that destination points to, using the respective folder the for key used.
It supports many keys and common cases, allowing to the developers to write
as little as possible. In this case, its output would be:

```bash
# rinstall
Installing "target/release/rinstall" to "/usr/local/bin/rinstall"
Installing "LICENSE.md" to "/usr/local/share/doc/rinstall-0.1/LICENSE.md"
Installing "README.md" to "/usr/local/share/doc/rinstall-0.1/README.md"
```

One of the projects that I stumbled upon which would benefit from using **rinstall** is the
widely used terminal emulator _[Alacritty]_. In addition to its static executable (Rust did it
again!), there is the man page, the example configuration, the `.desktop` file and its logo.
However, it lacks a `Makefile` and so the packagers (as well as the users) are required to
manually install these files. The Arch Linux [alacritty package][arch alacritty package]
contains the following instructions:

[arch alacritty package]: https://github.com/archlinux/svntogit-community/blob/packages/alacritty/trunk/PKGBUILD#L32

```bash
  desktop-file-install -m 644 --dir "$pkgdir/usr/share/applications/" "extra/linux/Alacritty.desktop"
  install -D -m755 "target/release/alacritty" "$pkgdir/usr/bin/alacritty"
  install -D -m644 "extra/alacritty.man" "$pkgdir/usr/share/man/man1/alacritty.1"
  install -D -m644 "extra/linux/io.alacritty.Alacritty.appdata.xml" "$pkgdir/usr/share/appdata/io.alacritty.Alacritty.appdata.xml"
  install -D -m644 "alacritty.yml" "$pkgdir/usr/share/doc/alacritty/example/alacritty.yml"
  install -D -m644 "extra/completions/alacritty.bash" "$pkgdir/usr/share/bash-completion/completions/alacritty"
  install -D -m644 "extra/completions/_alacritty" "$pkgdir/usr/share/zsh/site-functions/_alacritty"
  install -D -m644 "extra/completions/alacritty.fish" "$pkgdir/usr/share/fish/vendor_completions.d/alacritty.fish"
  install -D -m644 "extra/logo/alacritty-term.svg" "$pkgdir/usr/share/pixmaps/Alacritty.svg"
```

Its `install.yml` would be:

```
name: alacritty
version: 0.8.0
type: rust
exe:
    - src: alacritty
man:
    1:
      - extra/alacritty.man
completions:
    - bash: extra/completions/alacritty.bash
    - fish: extra/completions/alacritty.fish
    - zsh: extra/completions/_alacritty
data:
    - src: extra/logo/alacritty-term.svg
      dst: icons/hicolor/scalable/apps/Alacritty.svg
docs:
    - src: alacritty.yml
      dst: example/
appdata:
    - src: extra/linux/io.alacritty.Alacritty.appdata.xml
desktop-files:
    - src: extra/linux/Alacritty.desktopname: alacritty
```

And this would be the output from **rinstall**:

```bash
# rinstall
Installing "target/release/alacritty" to "/usr/local/bin/alacritty"
Installing "extra/logo/alacritty-term.svg" to "/usr/local/share/icons/hicolor/scalable/apps/Alacritty.svg"
Installing "extra/alacritty.man" to "/usr/local/share/man/man1/alacritty.1"
Installing "alacritty.yml" to "/usr/local/share/doc/alacritty-0.8.0/example/alacritty.yml"
Installing "extra/linux/Alacritty.desktop" to "/usr/local/share/applications/Alacritty.desktop"
Installing "extra/linux/io.alacritty.Alacritty.appdata.xml" to "/usr/local/share/appdata/io.alacritty.Alacritty.appdata.xml"
Installing "extra/completions/alacritty.bash" to "/usr/local/share/bash-completion/completions/alacritty.bash"
Installing "extra/completions/alacritty.fish" to "/usr/local/share/usr/share/fish/vendor_completions.d/alacritty.fish"
Installing "extra/completions/_alacritty" to "/usr/local/share/zsh/site-functions/_alacritty"
```

The original Arch Linux package contains 9 hardcoded install instructions that could be
replaced by 1 single line of a call to **rinstall**. That's 8 lines less. According to [repology],
alacritty is packaged for more than 30 different distributions. That's 240 lines less gained
by improving a single package! Yea, the improvement might actually be minor for other packages,
but it's still considerable, counting the thousand of packages out there!

[Alacritty]: https://github.com/alacritty/alacritty
[repology]: https://repology.org

## Makefiles comparison

I am sure many are asking why not add a `Makefile` for Alacritty instead? That's a great question.
In my opinion, these are the various reasons:

- its syntax is complicated and not easy to learn. Learning it as a requirement to install a simple
  shell script seems overkill to me.
- it is made to track dependencies between sources and objects and correctly compile software.
- not only [Directory Variables] standard is not enforced, the developer has to write support for it
  by himself. Asking the developer to learn an entire standard and the quirks of the
  [Filesystem Hierarchy Standard] seems again overkill.

[Filesystem Hierarchy Standard]: https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard#:~:text=The%20Filesystem%20Hierarchy%20Standard%20(FHS,released%20on%203%20June%202015.

While I was discussing **rinstall** with a developer, he sent me [this][fff Makefile]
`Makefile` to show me that they are really simple to write and that they support `MANDIR`
and `DOCDIR` without issues. Let's review its content:

[fff Makefile]: https://github.com/dylanaraps/fff/blob/master/Makefile

```makefile
    PREFIX ?= /usr
    MANDIR ?= $(PREFIX)/share/man
    DOCDIR ?= $(PREFIX)/share/doc/fff

all:
    @echo Run \'make install\' to install fff.

install:
    @mkdir -p $(DESTDIR)$(PREFIX)/bin
    @mkdir -p $(DESTDIR)$(MANDIR)/man1
    @mkdir -p $(DESTDIR)$(DOCDIR)
    @cp -p fff $(DESTDIR)$(PREFIX)/bin/fff
    @cp -p fff.1 $(DESTDIR)$(MANDIR)/man1
    @cp -p README.md $(DESTDIR)$(DOCDIR)
    @chmod 755 $(DESTDIR)$(PREFIX)/bin/fff

uninstall:
    @rm -rf $(DESTDIR)$(PREFIX)/bin/fff
    @rm -rf $(DESTDIR)$(MANDIR)/man1/fff.1
    @rm -rf $(DESTDIR)$(DOCDIR)
```

But, ironically, he sent me a Makefile with an error in it. Did you spot it?
I am sure that somebody did. For the rest, don't mind, it's actually a little (but crucial!)
detail: the default `PREFIX` is `/usr` instead of `/usr/local`. Not only this is wrong
according to the standard, but it opens the user to subtle breakage. The user could run `make install` as root and expect to have the newly installed files into `/usr/local`. He would then
to discover afterwards that it has silently installed into `/usr`. Only the package manager
should install into `/usr`, as it tracks all the files there. He may have
overwritten packages installed by the package manager and have potentially broken the system!

The equivalent `install.yml` is:

```yaml
name: fff
version: 0.1
type: custom
exe:
  - src: fff
man:
  1:
    fff.1
docs:
  - src: README.md
```

This was pretty a pretty subtle error to spot. More often the error will be easier to catch but
just as bad, such as a missing `DESTDIR`. I think it's not the developers mistake, as it is not fair
to ask them to write such fragile files following a standard that they might not even know that
well.

## Source based distributions

**rinstall** has been written in Rust. It is a language I have been learning recently and
that I fun to write into. It is also safe and fast, so I didn't give a second thought to use it
here. However, there have been an excellent critic regarding **rinstall** by a Gentoo maintainer:
does it mean that Rust toolchain will be needed by Gentoo stage3 when performing the base
installation?

**rinstall** aims at projects that are not already using a build system capable of installing
software. In other words, I don't want to replace Makefiles when they are already used
for compiling sources. This already exclude most of the software that needs to be compiled
in a stage3 or when bootstrapping. Moreover, having an `install.yml` does not enforce anyone to use
**rinstall**; the files could still be installed manually. In alternative, there will be a
release of **rinstall** available for anyone to grab; this will allow users to just download
the static executable of **rinstall** and use it.

## Conclusion

![](https://imgs.xkcd.com/comics/standards_2x.png)

This might as well be as introducing a new "standard" way to install files into the system.
However, I feel like an improvement in that regard is possible and that adding a new standard
is the only way to do so.

**rinstall** has been written in just a week and is actually missing some interesting use cases,
such as installing software from the release tarball. This is something that I am looking
forward to adding in the near future.

I want to thank everyone that has provided feedback on both **rinstall** and this post.
