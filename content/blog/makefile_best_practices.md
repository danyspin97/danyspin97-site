---
title: "Makefiles, Best Practices"
date: 2018-11-29T09:49:01+02:00
draft: false
---

`Makefile`s are widely used to build a lot of languages and projects, with C/C++ projects being the majority. Whenever you are developing or testing software, it's highly probable that you will encounter them.

This post will try to address some common errors in `Makefile`s, as well as good practices and cross-compiling support.

_**Prerequisites**: good understanding of `Makefile`s, UNIX directory hierarchy and compilation process._

## Variable setting

In this post we will use two ways (of the five available[^1]) to set a variable in `Makefile`. Here is a recap from StackOverflow[^2]:

Lazy Set

: Normal setting of a variable — values within it are recursively expanded when the variable is used, not when it's declared


```Makefile
VARIABLE = value
```

Immediate Set

: Setting of a variable with simple expansion of the values inside — values within it are expanded at declaration time.

```Makefile
VARIABLE := value
```

Set If Absent

: Setting of a variable only if it doesn't have a value

```Makefile
VARIABLE ?= value
```

## Compiler

```Makefile
CC ?= gcc
LD ?= gcc
```

While it is usually safe to assume that sensible values have been set for `CC` and `LDD`, it does no harm to set them if and _only_ if they are not already set in the environment, using the operator `?=`.

Using the assignment operator `=` will instead override `CC` and `LDD` values from the environment; it means that we choose the default compiler and it cannot be changed without editing the Makefile. This leads to two problems:

- The user has set `CC=clang` in the environment but `gcc` will be used anyway, even if it isn't installed.
- A cross-compile environment has set `CC` to a link of the actual target architecture compiler, like for example `arm-pc-linux-cc`, but `gcc` of the host will be used.

## Compiler flags

`make` utility also use variables that are defined by implicit rules[^3] and between these variables, some define extra build flags:

- `CFLAGS`: flags for the `C` compiler
- `CXXFLAGS`: flags for the `C++` compiler
- `CPPFLAGS`: for preprocessor flags for `C/C++` and `Fortran` compilers

_**Note**: there is a variable named `CCFLAGS` that some projects are using; it defines extra flags for both the `C`/`C++` compilers. This variable is not defined by the implicit rules, please avoid it if you can_.

_**Note 2**: build systems usually follow the `make` implicit rules for both variable naming and meaning. In other words, defining `CFLAGS` means defining extra flags for `C` compiler whatever build system you're using_.

Usually, we add to the compiler options specific for the application we're writing, such as language revision (do we want to use `c89` or `c99`?).
Then the user add his own `CFLAGS`/`CXXFLAGS` to include debug option and add optimizations; it is important to add these user defined flags.

We would be tempted to do:

```Makefile
CFLAGS = -ansi -std=99
```

But this would discard the user defined flags. You could instead do:

```Makefile
CFLAGS := ${CFLAGS} -ansi -std=99
```

or, if you have a long `CFLAGS`:

```Makefile
CFLAGS := ${CFLAGS}
CFLAGS += -ansi -std=99
```

**Note**: _we use the Immediate set (`:=`) because the Lazy set (`=`) would result in a recursive loop_.

## Libraries

In order to include the libraries in the
program, gcc flags are needed for both compile and link time.

You can add default values when including libraries, like for example default headers location `/usr/include/`; use this value inside a variable that can be overriden from the environment (`?=` set) if you follow this approach.

Suppose we want to include and link our program against `OpenSSL`, a broadly used library for cryptography and TLS/SSL protocols; we would intuitively add `-I/usr/include/openssl` to our `CFLAGS`/`CXXFLAGS`. It might be good for the most of the linux system, but a MacOS user could have OpenSSL headers in `/usr/local/include/openssl`, breaking compilation, same thing goes for cross compilation.

What it should be done instead is:

```makefile
OPENSSL_INCLUDE ?= -I/usr/include/openssl
OPENSSL_LIBS ?= -lssl -lcrypto

CFLAGS ?= -O2 -pipe
CFLAGS += $(OPENSSL_INCLUDE)
LIBS = $(OPENSSL_LIBS)
```

While this approach result in successful compilation, by overriding values when needed, it is really cumbersome and error-prone. A better way to include external libraries is to use `pkg-config`.

## pkg-config

`pkg-config`[^4] is a command-line tool that provides correct compiler options when including libraries; it is widely used in `Makefile`s but also in various build systems as `CMake`[^5] and `meson`[^6].

Let's rewrite the previous snippet using `pkg-config`:

```Makefile
PKG_CONFIG ?= pkg-config

CFLAGS ?= -O2 -pipe
CFLAGS += -std=99
CFLAGS += $(shell ${PKG_CONFIG} --cflags openssl)
LIBS := $(shell ${PKG_CONFIG} --libs openssl)
```

Note how `pkg-config` executable can be overriden from the environment, again for supporting cross-compiling.

Immediate set should be used with `LIBS` to avoid spawning `pkg-config` every time the variable is evaluated. _Thanks to u/dima55[^11] for the tip._

## Miscellaneous

Other executables often used when compiling are `ar`, `ranlib` and `as`; don't call these executables directly but
store their name in variables and use these variables instead.

```makefile
AR = ar
RANLIB = ranlib
AS = as
```
From `make` documentation[^7]:

> The precise recipe is `${AS} ${ASFLAGS}`.

## Installation

The last part of a `Makefile` is the installation of program itself and the related data. This is the trickiest part because there a lot of data types, and each one can be installed in various locations.

### PREFIX

Before talking about the different components to install, we should discuss briefly the `PREFIX` variable. The binaries usually go in a `bin` directory, in Linux environment `/usr/bin` is used for system packages managed by the package managers and `/usr/local/bin` for system packages installed by the user, FreeBSD ports use `/usr/local/bin`; same goes for the other components (data, man pages).

It is important to specify which `PREFIX` the `bin` directory should be installed in:

```makefile
PREFIX ?= /usr
```

You can either choose `/usr` and `/usr/local`.


### BINDIR

The principal component of every program (excluding libraries) is the executable itself. As we stated before, the usually directory is `${PREFIX}/bin`, so at a first glance we could choose to install the executables directly there like this:

```makefile
install:
    @cp -p foo ${PREFIX}/bin/foo
```

While this is correct most of the time, it limits flexibility of the `Makefile` we're writing. To give user more choice we should instead use a variable to specify executables location: `BINDIR`.

```makefile
BINDIR ?= ${PREFIX}/bin

install:
    @cp -p foo ${BINDIR}/foo
```

### DATADIR

From the GNU Coding Standards[^8]:

> The directory for installing idiosyncratic read-only architecture-independent data files for this program. 

`DATADIR` directory, usually `/usr/share` contains read-only data, from application icon to man pages and documentation.

```makefile
DATADIR ?= ${PREFIX}/share
```
It is important to specify `DATADIR` because `MANDIR` and other variables depend on it.

It can happen that this directory doesn't start with `PREFIX`; for example a cross-compiled system can have the executables in `/usr/${ARCH}/bin` (therefore `/usr/${ARCH}` as `PREFIX`) and have `/usr/share` as `DATADIR`.

### MANDIR

The location where the man pages are installed is stored in `MANDIR`.

If you are using `DATADIR`:

```
MANDIR ?= ${DATADIR}/man
```

Otherwise:

```makefile
MANDIR ?= ${PREFIX}/share/man
```

The first approach is better because a user can just set `DATADIR` and forget about `MANDIR`; they are equally flexible.

### DESTDIR

Now, we chose all the various component location, there is one last important question to ask ourself: _which is the base install directory_?

We usually want to install the program in root folder which works with everything we've seen so far. Let's suppose there is a system mounted in whatever place or the toolchain merges the a special destination directory with the root install, how can we target this custom destination directory?

`DESTDIR` is a special variable that let us specify a destination directory. Its usage is simple, we prefix all the variables we saw with `${DESTDIR}` in the `install` section.

```makefile
install:
    @cp -p foo ${DESTDIR}${BINDIR}/foo
    @cp -p foo ${DESTDIR}${MANDIR}/man1
```

There is no need to set it because either the user set `DESTDIR` or `/` will be used.

Note that the following assignment to `BINDIR` is incorrect because `DESTDIR` should only be added later in the `install` section and not on variable assignment.

```makefile
BINDIR = ${DESTDIR}${PREFIX}/bin
```

### Other locations

Since there are many other locations, we covered the most used ones. Refer to GNU Coding Standard[^8] and big projects' `Makefile` to see other variables.

## Reproducible builds

Another practice, which got an increasing usage in the last years, is _reproducible builds_[^9]:

> **Reproducible builds** are a set of software development practices that create an **independently-verifiable path** from **source code to the binary code** used by computers.

This practice is being adopted by Debian[^10] and it is worth adopting in general. Explaining how to adopt _reproducible builds_ goes beyond the scope of this post, but you can read more in the official site[^9].

## Conclusion

Following these rules and standards you will have a better `Makefile`, along with great flexibility among all the user environments and cross-compiling support.

[^1]: https://www.gnu.org/software/make/manual/make.html#Setting
[^2]: https://stackoverflow.com/a/448939
[^3]: https://www.gnu.org/software/make/manual/make.html#index-CFLAGS
[^4]: https://www.freedesktop.org/wiki/Software/pkg-config/
[^5]: https://cmake.org
[^6]: https://mesonbuild.com
[^7]: https://www.gnu.org/software/make/manual/make.html#index-AR
[^8]: https://www.gnu.org/prep/standards/html_node/Directory-Variables.html
[^9]: https://reproducible-builds.org/
[^10]: https://wiki.debian.org/ReproducibleBuilds
[^11]: https://www.reddit.com/user/dima55
