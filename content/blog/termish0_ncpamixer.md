---
title: "Termish[0] = ncpamixer"
date: 2018-02-05T14:40:52+01:00
categories: ["termish", "audio"]
tags: ["terminal", "pulseaudio", "pavucontrol"]
draft: false
---

Welcome to the first post of *Termish*. Please, read what this is about
[here](/blog/termish_malloc/), if you haven't already.

Let's talk about _PulseAudio_, a popular sound server that is widely used in
Linux Desktop for its features out of the box. If you don't know which your
system audio service is, probably there is PulseAudio running (considering
that even Firefox use it as default on Linux from version [52][Firefox 52]).

How can we control audio input/output and raise volume? There is a graphical
interface, _pavucontrol_, which accomplish those easily.

<br/>

![pavucontrol](/img/pavucontrol.png)

<br/>

Is there a lightweight alternative to pavucontrol? Something like [alsamixer]
to use from our beloved terminal? Well, yes; its name is _[ncpamixer]_.

ncpamixer, which stands for _ncurses PulseAudio Mixer_, offers basic
PulseAudio server control from a terminal interface; even more, there is a
customizable shortcut for anything.

<br/>

![ncpamixer](/img/ncpamixer.gif)

<br/>

At a first glance it really seems to use pavucontrol from the terminal, in
fact the README says:

> An ncurses mixer for PulseAudio inspired by pavucontrol.

# Features

_ncpamixer_ implements the following features:

- _per_ application volume changing
- devices monitoring and configuration
- granular input and output control

However, it cannot do (but _pavucontrol_ can):

- change latency offset
- do any advanced configuration
- filter applications by category

# Installation

If you want to give it a try, [these][install] are the installation instructions.

# Conclusion

_ncpamixer_ is not a drop-in replacement for its GTK+ counterpart, but it's
suitable for everyday usage: it is perfect to use along with a window manager
and a shortcut assigned (I personally use super+p).

[Firefox 52]: https://www.bleepingcomputer.com/news/software/some-firefox-52-users-on-linux-left-without-sound/
[alsamixer]: https://en.wikipedia.org/wiki/Alsamixer
[ncpamixer]: https://github.com/fulhax/ncpamixer
[install]: https://github.com/fulhax/ncpamixer#install
