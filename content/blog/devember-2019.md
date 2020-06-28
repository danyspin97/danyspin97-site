---
title: "Devember 2019: Rewriting 66"
date: 2019-11-25T15:15:00+01:00
categories: [development]
tags: ['exherbo', 'init-diversity', 'devember', 'dlang']
---

**Prerequisites**: _Basic knowledge of a Unix system architecture; init diversity initiative._

_Devember_[^1] it's coming and for this year (which is also my first year
participating) I've chosen something really close to me: the init and service
manager (_called init/rc for the rest of the post_). Specifically I've chosen to
**rewrite 66 from scratch**.

**Notes**: _In the following paragraphs there is an explanation of what led me
to rewrite 66. I was planning to explain this from some time and I've taken to
opportunity to do so now. If you are only interested in the program itself and
what I'll do in this Devember, skip to [development](#tt)._

## Why?

Regardless of whatever “sytemd sucks”[^3][^4] or not, I want to have an
alternative to it. I want to be able to have at least some choiches for what
runs on my system, especially when we talk about critic programs such as **PID
1** and the service manager.

The alternatives currently availables (_OpenRC_, _runit_) do not offer the
advantages of _systemd_ or have some big disadvantage that do not make their use
straightforward. This does not involve the design (which have been already
covered in deep by _skarnet_ here[^5]) but only usability. Yup, I am really
picky regarding the programs to use.

## s6/s6-rc

At the start of the 2018 I've learnt about s6 suite[^9] and s6-rc[^10]. I wanted
to try them on my newly installed _Exherbo Linux_ system, so I have adapted the
example services to run on my machine. Thanks to the help of the **#s6**
official IRC channel the system finally run s6/s6-rc and it was *great*; at boot
it started all the services asynchronously and showed a working tty in the blink
of an eye.

I've liked it so much that I stared contributing to the integration of s6 into
exherbo. It consisted in **s6-rc services for the various packages and a sane
set of starting scripts** to have a working system (called _s6-exherbo_[^11]).
My small VPS I configured at the time even got s6 on it.

With the time passing I've found the **service writing and administration** to
be really **time consuming**. And that's because it has not a user friendly
interface, thus it was not made for what me (and some other users) were using
it. And here it comes *s6-frontend*, the user interface not written yet. Skarnet
has recently confirmed that he will write it in the 2021 so it won't be ready
any time soon.

## 66

Exherbo was not the only distribution adopting s6 and and among the other ones
there was **obarun**[^13], developed by the omonimus creator as a fork of Arch
Linux. He, too, wrote a lot of wrapper scripts to ease the use of the s6 suite,
but in the end he resorted to writing his own frontend, called **66**[^14].
Obarun (_the distribution_) was converted for 66 as soon as a working release got out.

On the other hand there was me who became an early adopter the moment I looked
at the documentation[^14]: **it simply resolves almost all my issues of s6/s6-rc** without any major disadvantage. It is **easy to use**, **powerful** and **extensible**.

So far, s6-exherbo got immediately forked into _66-exherbo_[^12] and the old services converted to the new frontend format. Devuan too got his own
_66-devuan_[^15], which took me more than it should have had. If you are curious
about the reason, 66's developer did not want to support FHS[^6] in his general
set of starting scripts, because he did not like this standard. In the end he
accepted my patches and **Devuan booted using 66**.

Service enabling on Exherbo with system version of 66 suffered from a bug
(_Hello SIGSEGV, long time no see_) and it was unusable; to make the matter
worse, the bug could not be reproduced with a local copy. The developer had
something else on his schedule (writing a new command-line arguments parser,
apparently) and did not want to fix the bug. “**I can fix it myself**”, I
thought: _I was wrong_.

66 codebase consists of **16000 lines with little to no comments**, and it makes
heavy use of skalibs[^16] and oblibs[^17] libraries, greatly lowering the
readability (I'd like to say it is written in _skarnet's C_). Downgrade wasn't
an option neither due to a breaking change in services and the various services
in Exherbo already got updated. I could not find the bug, I could not enable new
services and the developer did not care to fix this fatal bug. Devuan too was
failing to build 66. **There was enough reasons for me to fork the project** and
improve it, adding unit testing and have better code quality.

Now that I had the possibility, I could also **rewrite it from scratch**, picking different choiches from the start and _avoiding the legacy code_.

**Note**: _I really like 66 project but it simply isn’t what I am searching for.
I wish the best to obarun with both 66 and his distribution, for which he have
worked years trying to offer a valid systemd and Arch Linux alternative_.

## tt

**tt** (which should have been 77 or t7 but I don't like numbers in binaries
names, if not strictly necessaries) is a wrapper, or better, **a frontend to
s6/s6-rc**.

A bit about its development:

Written in **D**

: The suited languages for such a project are `C`, `C++`, `Rust` and `D`. _C_ is
the fastest if used correctly; but it's really hard to get right and requires
developers to rewrite a lot of stuff (or use libs like glib or skalibs). _C++_
is hard and share some problem with _C_, I prefer to not use if I have a
choiche. _Rust_ has a microdependency ecosystem and it is hard to use C
libraries as well as exposing C bindings. On the other hand, _D_ seems suited
for such a project: it is **safer than C**, **has a GC**, and it is relatively
**fast to write**, while keeping a good performance and high flexibility.

Use C or C++ libs

: D community is not very active, so many libraries have been abandoned and the
maintained ones could still have the same ending in the future. To avoid such a
possibility, **I have chosen to only use C/C++ libraries** (way more _tested_
and _actively maintained_), since Dlang makes them easy to use.

Provide external C bindings

: tt will have a library and a command-line interface. In the long-term there
could be additional command lines interfaces, plugins or GUI programs to
administrate the system from; it is important for me to **expose a C interface
so that any language could be used** (after all almost every language permits
to call C functions).

Try to not reinvent the wheel

: To get a working service manager as soon as possible, **complex tasks will be
achieved using external libraries** (like parsing the _services files_).
Reinventing the wheel is good for learning purposes, but increase the
development and testing time too much for my tastes. Therefore I will try to
keep it to a minimum.

Features

: These are 66[^14]’s features, which **tt** will keep closely to.

> - Frontend service files declaration.
- Backup a complete set of services.
- Easy creation of a scandir.
- Nested supervision tree.
- Instance service file creation.
- Multiple directories service file declaration(packager,sysadmin,user).
- Easy change of service configuration.
- Automatic logger creation. 

Sane defaults

: The most important thing is to **make the pc boot, whatever the conditions**.
The users and the distribution maintainers should do the least work possible.
Providing sane defaults and sane fallback helps in this matter. For example
`stage1` should work regardless if the initramfs has been used or not (and _this
didn’t happen in 66 until recently_).

About the libaries used, I have to try them so I’ll problably post more details in the next weeks.

[^1]: https://devember.org/
[^3]: https://skarnet.org/software/systemd.html
[^4]: https://suckless.org/sucks/systemd/
[^5]: https://skarnet.org/software/s6/why.html
[^6]: https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard
[^9]: https://skarnet.org/software/s6/
[^10]: https://skarnet.org/software/s6-rc/
[^11]: https://gitlab.exherbo.org/exherbo-misc/s6-exherbo
[^12]: https://gitlab.exherbo.org/exherbo-misc/66-exherbo
[^13]: https://web.obarun.org/
[^14]: https://web.obarun.org/software/66/
[^15]: https://git.devuan.org/66-devuan
[^16]: https://skarnet.org/software/skalibs/
[^17]: https://framagit.org/obarun/oblibs
